---
title: Database Schema Drift
impact: HIGH
impactDescription: "Production schema diverges from code; migrations break unpredictably"
tags: database, migrations, schema
---

## Database Schema Drift

**Impact: HIGH (Production schema diverges from code; migrations break unpredictably)**

Schema drift is the gap between what the migrations say the database looks like and what it actually looks like. Once it exists, every new migration is a roll of the dice — it might apply cleanly, fail halfway through, or succeed but leave the schema in a state neither environment expects.

## How to Detect

```bash
# Laravel: compare current schema to migration plan
php artisan migrate:status
php artisan schema:dump --prune          # snapshot current schema
# Apply on a fresh DB and diff against production schema

# MySQL: schema-only dump for diffing across environments
mysqldump --no-data --routines --triggers -u root -p prod_db    > prod.sql
mysqldump --no-data --routines --triggers -u root -p staging_db > staging.sql
diff -u staging.sql prod.sql

# Drift signals:
# - Tables that exist in prod but not in any migration
# - Columns/indexes added manually via ALTER TABLE outside migration
# - Migrations marked "ran" in different orders across environments
# - Migrations that have been edited in place (hashes differ)
```

Tooling: `atlas migrate diff`, `bytebase`, `liquibase diff`, `flyway info` for managed migration platforms.

## Incorrect

```
❌ Common drift scenarios:

1. "Quick fix" applied directly to prod
   DBA runs:  ALTER TABLE orders ADD COLUMN priority INT DEFAULT 0;
   …without a corresponding migration. New migrations now run against a schema
   the codebase has never seen.

2. Migration edited after being applied somewhere
   Original: CREATE TABLE refunds (id, order_id, amount);
   Edited:   CREATE TABLE refunds (id, order_id, amount, reason TEXT NOT NULL);
   Some environments have the column; others don't. Same migration hash, two realities.

3. Models with attributes not in any migration
   class Order extends Model {
     protected $fillable = ['status', 'priority'];  // 'priority' has no migration
   }
```

**Problems:**
- New environments (CI, staging, new dev laptops) can't reach the same schema
- The next migration may fail at 50% completion, leaving the schema half-done
- Reports and queries silently rely on columns that "are there in prod" but not in code

## Correct

```php
// ✅ Every schema change goes through a migration
// One migration per change, never edit a merged migration

// database/migrations/2026_05_16_000000_add_priority_to_orders.php
return new class extends Migration {
    public function up(): void {
        Schema::table('orders', fn (Blueprint $t) =>
            $t->unsignedTinyInteger('priority')->default(0)->after('status')
        );
    }
    public function down(): void {
        Schema::table('orders', fn (Blueprint $t) => $t->dropColumn('priority'));
    }
};
```

CI gate:

```yaml
- name: Schema is migration-derivable
  run: |
    php artisan migrate --pretend --database=ci_clone   # ensure all migrations apply
    php artisan schema:dump --prune
    git diff --exit-code database/schema/               # fail if dump differs
```

**Benefits:**
- Any environment can be reconstructed from migrations alone
- Code and schema move together, atomically reviewable in PRs
- A failed migration in CI catches drift before it hits prod

## Remediation Strategy

- **Effort:** M–L (depends on how far drift has progressed)
- **When to pay down:**
  - **NOW:** any drift discovered during incident response — fix during the postmortem
  - **As a cleanup project:** snapshot prod schema → generate a "consolidation migration" that brings empty databases to current state → mark all prior migrations as "ran" in environments that already match
- **Anti-patterns:**
  - Editing applied migrations (always create a new one)
  - "DBA runs prod ALTERs directly" without a corresponding migration
  - Squashing migrations in a way that breaks existing environments

**Tip:** in long-lived projects, periodically generate a "consolidated migration" from the current schema (`schema:dump`) so new environments don't have to replay 5 years of migrations. Keep the consolidated dump and historical migrations both checked in.

Reference: [Laravel — Schema Dumping](https://laravel.com/docs/migrations#squashing-migrations) · [Atlas — Migration Diff](https://atlasgo.io/) · [Liquibase Diff](https://docs.liquibase.com/commands/inspection/diff.html)
