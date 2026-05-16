---
title: Failed-Job Storage and Retention
impact: HIGH
impactDescription: "Failed jobs are your last line of evidence; lose them and you lose the incident"
tags: config, failed-jobs, retention
---

## Failed-Job Storage and Retention

**Impact: HIGH (Failed jobs are your last line of evidence; lose them and you lose the incident)**

When a job has burned all `$tries`, Laravel writes a row to the `failed_jobs` table (or the configured `failed.driver`). This is the *only* persistent record of what went wrong. Without it, you're blind during incidents. Without retention/cleanup, the table grows unbounded.

Three things need to be right:

1. **Storage configured** — `failed_jobs` table exists (Laravel scaffolds the migration) or DynamoDB driver is configured for Vapor/serverless
2. **Retention policy** — `queue:prune-failed` scheduled, OR exported to long-term storage (Sentry, Datadog) and pruned aggressively
3. **Alerting** — somebody sees the failures (alert when `failed_jobs` count grows; don't rely on logs alone)

## Incorrect — common misses

```php
// ❌ The failed_jobs table doesn't exist
// (forgot to run the migration that ships with Laravel)
// $ php artisan migrate --pretend
// → no create_failed_jobs_table migration found
// Result: failed jobs vanish into the error log; no record, no retry.

// ❌ DynamoDB driver configured without the table actually existing
// config/queue.php:
'failed' => [
    'driver' => env('QUEUE_FAILED_DRIVER', 'dynamodb'),
    // …
    'table' => 'failed_jobs',
],
// → DynamoDB table 'failed_jobs' was never created in the AWS account.
// Failures silently dropped.

// ❌ Table grows forever
// failed_jobs has 4.2 million rows from 18 months. Nobody noticed.
// The table now slows DB backups; nobody dares prune it manually.
```

## Correct

### Storage — database driver (default)

```bash
# Laravel ships the migration; ensure it ran
php artisan migrate
```

The migration creates:
```sql
CREATE TABLE failed_jobs (
    id bigint unsigned not null auto_increment primary key,
    uuid varchar(255) unique,
    connection text not null,
    queue text not null,
    payload longtext not null,
    exception longtext not null,
    failed_at timestamp not null default current_timestamp
);
```

### Storage — DynamoDB driver (Vapor / serverless)

```php
// config/queue.php
'failed' => [
    'driver' => env('QUEUE_FAILED_DRIVER', 'dynamodb'),
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'failed_jobs',
],
```

Then create the DynamoDB table:

```bash
aws dynamodb create-table \
  --table-name failed_jobs \
  --attribute-definitions AttributeName=application,AttributeType=S AttributeName=uuid,AttributeType=S \
  --key-schema AttributeName=application,KeyType=HASH AttributeName=uuid,KeyType=RANGE \
  --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
```

### Retention — schedule pruning

```php
// app/Console/Kernel.php  (Laravel 13: routes/console.php)
use Illuminate\Console\Scheduling\Schedule;

Schedule::command('queue:prune-failed --hours=720')  // 30 days
    ->daily();
```

`queue:prune-failed` works on both database and DynamoDB failed-job stores.

### Alerting — surface the failures

A `failed_jobs` row sitting in the table that nobody sees is debt. Wire alerts:

```php
// app/Providers/AppServiceProvider.php
use Illuminate\Queue\Events\JobFailed;
use Illuminate\Support\Facades\Event;
use Sentry\Laravel\Integration;

public function boot(): void
{
    Event::listen(function (JobFailed $event) {
        // Send to Sentry / Bugsnag / paging system
        report($event->exception);
        // Or: Slack / email a daily digest from a scheduled command
    });
}
```

Alternatively, schedule a daily check that fails the build if `failed_jobs` grew unexpectedly:

```bash
# In a CI cron or healthcheck:
COUNT=$(php artisan tinker --execute='echo DB::table("failed_jobs")->count();')
if [ "$COUNT" -gt 100 ]; then
  echo "ALERT: failed_jobs has $COUNT rows"; exit 1
fi
```

## Detection

```bash
# Failed-jobs storage configured?
grep -A5 "'failed'" config/queue.php

# failed_jobs migration present?
ls database/migrations/*create_failed_jobs*

# Is queue:prune-failed scheduled?
grep -rE 'queue:prune-failed' app/Console/ routes/console.php 2>/dev/null

# Current size (database driver)
php artisan tinker --execute='echo DB::table("failed_jobs")->count();'
```

Reference: [Laravel 13 — Queues: Storing Failed Jobs in DynamoDB](https://laravel.com/docs/13.x/queues#storing-failed-jobs-in-dynamodb) · [Laravel 13 — Queues: Cleaning Up After Failed Jobs](https://laravel.com/docs/13.x/queues#cleaning-up-after-failed-jobs)
