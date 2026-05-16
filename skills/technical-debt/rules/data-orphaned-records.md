---
title: Orphaned Records and Referential Drift
impact: MEDIUM
impactDescription: "Bugs, broken reports, and inconsistent state compound over time"
tags: database, referential-integrity, data-quality
---

## Orphaned Records and Referential Drift

**Impact: MEDIUM (Bugs, broken reports, and inconsistent state compound over time)**

Orphaned records — child rows whose parent no longer exists — are usually invisible until a report breaks or a query joins fail mysteriously. The root cause is almost always missing foreign-key constraints, ad-hoc deletes that bypass the ORM, or "soft-delete the parent, leave children active" semantics.

## How to Detect

```sql
-- Orphans on a single relation
SELECT child.id, child.parent_id
FROM order_items child
LEFT JOIN orders parent ON parent.id = child.parent_id
WHERE parent.id IS NULL;

-- MySQL: columns ending in _id that are NOT part of any FK constraint.
-- Note: `_` is a single-char wildcard in LIKE — escape it (use `#` as ESCAPE char to
-- avoid backslash-escape confusion in MySQL string literals). Exclude the PK `id` column.
SELECT c.TABLE_NAME, c.COLUMN_NAME
FROM information_schema.COLUMNS c
WHERE c.TABLE_SCHEMA = DATABASE()
  AND c.COLUMN_NAME LIKE '%#_id' ESCAPE '#'
  AND c.COLUMN_NAME <> 'id'
  AND NOT EXISTS (
    SELECT 1
    FROM information_schema.KEY_COLUMN_USAGE kcu
    JOIN information_schema.TABLE_CONSTRAINTS tc
      ON tc.CONSTRAINT_NAME   = kcu.CONSTRAINT_NAME
     AND tc.CONSTRAINT_SCHEMA = kcu.CONSTRAINT_SCHEMA
     AND tc.TABLE_NAME        = kcu.TABLE_NAME
    WHERE tc.CONSTRAINT_TYPE   = 'FOREIGN KEY'
      AND kcu.TABLE_SCHEMA     = c.TABLE_SCHEMA
      AND kcu.TABLE_NAME       = c.TABLE_NAME
      AND kcu.COLUMN_NAME      = c.COLUMN_NAME
  );

-- Soft-deleted parents with active children
SELECT COUNT(*) FROM orders
WHERE deleted_at IS NOT NULL
  AND EXISTS (
    SELECT 1 FROM order_items
    WHERE order_id = orders.id AND deleted_at IS NULL
  );
```

## Incorrect

```php
// ❌ No FK constraints; manual delete bypasses cascades
Schema::create('order_items', function (Blueprint $t) {
    $t->id();
    $t->unsignedBigInteger('order_id');          // no constraint, no index
    $t->unsignedBigInteger('product_id');
});

// Deletion via raw SQL — children become orphans
DB::table('orders')->where('status', 'cancelled')->delete();
// 200 order rows gone; their 1,800 order_items now point to nothing.

// Soft-delete parent, hard-active children — reports show ghost orders
class Order extends Model { use SoftDeletes; }
class OrderItem extends Model {}   // does NOT respect parent's deleted_at
```

**Problems:**
- Joins return rows but with `NULL` parents, silently breaking sums and counts
- Cleanup jobs run forever ("delete order_items with no order" finds millions)
- Restore-from-backup workflows can't trust the data they restore

## Correct

```php
// ✅ FK constraints with explicit cascade behaviour
Schema::create('order_items', function (Blueprint $t) {
    $t->id();
    $t->foreignId('order_id')
        ->constrained()                     // creates FK to orders(id)
        ->cascadeOnDelete();                // delete items when order is deleted
    $t->foreignId('product_id')
        ->constrained()
        ->restrictOnDelete();               // can't delete a product still referenced
});

// ✅ Soft-deletes coordinated across parent and children
//    Both models must use SoftDeletes for soft-cascade to work (otherwise
//    `$order->items()->delete()` will hard-delete the children).
class OrderItem extends Model {
    use SoftDeletes;
}

class Order extends Model {
    use SoftDeletes;
    protected static function booted() {
        static::deleted(fn($order) => $order->items()->delete());      // soft-deletes children
        static::restored(fn($order) => $order->items()->withTrashed()->restore());
    }
}
```

```sql
-- ✅ Add missing FKs to legacy tables (MySQL/InnoDB; clean orphans first)
-- Note: adding a FK with `ALGORITHM=INPLACE` is only supported when
-- `foreign_key_checks=OFF`, and `LOCK=NONE` is NOT supported for FK adds.
-- With default settings MySQL forces `ALGORITHM=COPY`, which rewrites
-- the table. For large tables, use `pt-online-schema-change` or `gh-ost`
-- to add the FK without blocking writes.
ALTER TABLE order_items
  ADD CONSTRAINT fk_order_items_order
  FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE;
```

**Benefits:**
- Database enforces integrity even when application code is buggy
- Reports and joins are trustworthy
- Cleanup jobs become unnecessary (or trivial)

## Remediation Strategy

- **Effort:**
  - **S** — adding constraints to greenfield tables
  - **M** — backfilling constraints + cleanup on small/medium tables
  - **L** — adding constraints to a 100M+ row table (must clean orphans first, then add constraint online)
- **When to pay down:**
  - **NOW:** any data integrity issue traced back to orphans
  - **As cleanup project:** audit tables for `_id` columns without FK constraints
  - **Then:** make FKs required for all new tables (lint a migration template)

**Cleanup workflow:**
1. Run the orphan-detection query above for each suspected relation
2. Decide policy: hard-delete orphans, mark them archived, or attach to a placeholder parent
3. Apply the cleanup in batches (avoid one giant transaction)
4. Add the FK constraint
5. Add a CI test that runs the orphan-detection query against the test DB after each test run

Reference: [MySQL — FOREIGN KEY Constraints](https://dev.mysql.com/doc/refman/8.4/en/create-table-foreign-keys.html) · [Laravel — Foreign Key Constraints](https://laravel.com/docs/migrations#foreign-key-constraints)
