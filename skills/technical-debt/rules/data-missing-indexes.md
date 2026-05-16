---
title: Missing Database Indexes
impact: HIGH
impactDescription: "Query time grows linearly with data; locks and connection pool compound"
tags: database, indexes, performance
---

## Missing Database Indexes

**Impact: HIGH (Query time grows linearly with data; locks and connection pool compound)**

A missing index turns a 5ms query into a 5-second query as the table grows. Worse, because slow queries hold connections longer, missing indexes cascade into connection-pool exhaustion and 503s for unrelated traffic. Indexing decisions made early are usually right; indexes never added at all are silent debt.

## How to Detect

```sql
-- MySQL (requires sys schema, enabled by default in 5.7+): tables doing frequent full scans
SELECT * FROM sys.schema_tables_with_full_table_scans
WHERE rows_full_scanned > 1000
ORDER BY rows_full_scanned DESC;

-- MySQL: query plan for a hot query (type=ALL means full table scan)
EXPLAIN     SELECT * FROM orders WHERE customer_id = 123;
EXPLAIN ANALYZE  SELECT * FROM orders WHERE customer_id = 123;   -- MySQL 8.0+

-- MySQL: indexes that exist but are never read
SELECT * FROM sys.schema_unused_indexes;

-- MySQL: top queries by total latency (performance_schema must be ON)
SELECT * FROM sys.statement_analysis
ORDER BY total_latency DESC LIMIT 20;
```

Application-side:
- **Laravel:** Telescope's "Queries" panel — sort by duration
- **Node ORMs:** enable query logging (Prisma `log: ['query', 'warn']`, TypeORM `logging: 'all'`) and review slow entries

Look for: queries on foreign keys, status columns, date filters, and `ORDER BY` columns without supporting indexes.

## Incorrect

```php
// ❌ Foreign key columns without indexes
Schema::create('orders', function (Blueprint $t) {
    $t->id();
    $t->unsignedBigInteger('customer_id');     // FK column — no index!
    $t->string('status');                       // queried often — no index!
    $t->timestamp('created_at');
});

// Query: SELECT * FROM orders WHERE customer_id = ? AND status = 'paid' ORDER BY created_at DESC
//   → Seq Scan of the entire orders table for every customer profile load
```

**Problems:**
- Customer-profile page becomes O(N) of total orders, not the customer's orders
- Status-based dashboards lock the table during long scans
- Connection pool saturates under modest load

## Correct

```php
// ✅ Index foreign keys, status columns, and ORDER BY columns
Schema::create('orders', function (Blueprint $t) {
    $t->id();
    $t->foreignId('customer_id')->constrained()->index();    // index on FK
    $t->string('status')->index();
    $t->timestamp('created_at');

    // Composite index for the common (customer_id, status, created_at) query path
    $t->index(['customer_id', 'status', 'created_at']);
});
```

Verify the plan after indexing:

```sql
EXPLAIN
SELECT * FROM orders
WHERE customer_id = 123 AND status = 'paid'
ORDER BY created_at DESC LIMIT 50;

-- Want to see in `key`: orders_customer_id_status_created_at_index
-- NOT:                  NULL  (or `type` = ALL → full table scan)
```

**Benefits:**
- Query time becomes O(log N) instead of O(N)
- Connection pool stays healthy under load
- The index pays for itself many times over per request

## Remediation Strategy

- **Effort:**
  - **S** — add a single index (online index creation in MySQL/InnoDB minimizes downtime)
  - **M** — add several indexes; analyze query patterns first
  - **L** — large tables (100M+ rows) require careful online build + monitoring
- **When to pay down:**
  - **NOW:** any query on a hot path showing `type=ALL` in EXPLAIN over a > 10k-row table
  - **NOW:** any FK column without an index (defaults vary by ORM — Eloquent does not auto-index FKs)
  - **Then:** monitor `sys.statement_analysis` weekly and add indexes for the top long-runners

**Anti-patterns:**
- **Indexing everything** — wastes write speed and disk; index the columns you actually filter/sort by
- **Adding indexes blindly** without `EXPLAIN ANALYZE` — verify the planner actually uses them
- **Forgetting to remove old indexes** — duplicate or unused indexes cost disk + write speed

**Online index creation (MySQL/InnoDB):**
```sql
ALTER TABLE orders
  ADD INDEX orders_customer_status_created_at_index (customer_id, status, created_at),
  ALGORITHM=INPLACE, LOCK=NONE;
```

For huge tables, prefer `pt-online-schema-change` (Percona Toolkit) or `gh-ost` (GitHub) — both run truly non-blocking schema changes.

Reference: [Use the Index, Luke](https://use-the-index-luke.com/) · [MySQL — EXPLAIN Output Format](https://dev.mysql.com/doc/refman/8.4/en/explain-output.html) · [MySQL — sys Schema](https://dev.mysql.com/doc/refman/8.4/en/sys-schema.html) · [pt-online-schema-change](https://docs.percona.com/percona-toolkit/pt-online-schema-change.html)
