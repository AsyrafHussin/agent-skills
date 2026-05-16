---
title: Queue Driver Choice — database vs redis vs sqs
impact: CRITICAL
impactDescription: "Wrong driver at scale is the bottleneck; right driver scales until you don't notice it"
tags: config, driver, redis, database, sqs
---

## Queue Driver Choice — database vs redis vs sqs

**Impact: CRITICAL (Wrong driver at scale is the bottleneck; right driver scales until you don't notice it)**

Laravel 13's queue config supports `sync`, `database`, `redis`, `sqs`, `beanstalkd`, `deferred`, `background`, `failover`, and `null`. For most production Laravel + MySQL apps, the realistic choices are **`database`**, **`redis`**, or — on Laravel Vapor — **`sqs`**. The `deferred`/`background`/`failover`/`null` drivers serve niche cases (in-request deferral, request-disabled async, fall-through routing, testing) and aren't covered further here. Pick wrong on the main three and you spend months fighting symptoms.

## When to use each

| Driver | Use when | Avoid when |
|---|---|---|
| **`sync`** | Local dev only (runs jobs inline, ignoring `ShouldQueue`) | Anywhere in production |
| **`database`** | Small apps (< 10k jobs/day), simple setup, no Redis available | High throughput (lock contention on `jobs` table); Laravel 11+ uses `SKIP LOCKED` which helps but doesn't eliminate the issue |
| **`redis`** | Production scale — sub-millisecond pop, `block_for` blocking polling, Horizon support | No Redis in the stack (forces unnecessary infra) |
| **`sqs`** | Laravel Vapor / AWS-native deployments | Self-hosted; the AWS round-trip latency is higher than Redis |

The default for new Laravel 13 apps is `database` — fine to start, **plan to migrate to `redis` before traffic warrants it.**

## Incorrect

```php
// ❌ Using sync in production "for simplicity"
// .env:
QUEUE_CONNECTION=sync

// Result: every dispatched job runs inline, blocking the HTTP request.
// ShouldQueue does nothing. A "queued" email send blocks checkout by 3 seconds.
```

```php
// ❌ Sticking with database driver at scale
// .env:
QUEUE_CONNECTION=database

// At 100 req/s peak, the `jobs` table sees:
//   - INSERT on dispatch (every job)
//   - SELECT ... FOR UPDATE on pop (every worker tick)
//   - UPDATE/DELETE on completion
// MySQL is the bottleneck. Latency rises; jobs back up.
```

## Correct

```php
// ✅ Production: Redis with sensible defaults

// config/queue.php
'connections' => [
    'redis' => [
        'driver' => 'redis',
        'connection' => env('REDIS_QUEUE_CONNECTION', 'default'),
        'queue' => env('REDIS_QUEUE', 'default'),
        'retry_after' => env('REDIS_QUEUE_RETRY_AFTER', 90),
        'block_for' => 5,                 // blocking pop (framework default is `null`); worker idles cheaply
        'after_commit' => true,           // see config-after-commit rule
    ],
],

// .env:
QUEUE_CONNECTION=redis
```

```php
// ✅ Small app or staging: database
// config/queue.php
'connections' => [
    'database' => [
        'driver' => 'database',
        'connection' => env('DB_QUEUE_CONNECTION'),
        'table' => env('DB_QUEUE_TABLE', 'jobs'),
        'queue' => env('DB_QUEUE', 'default'),
        'retry_after' => env('DB_QUEUE_RETRY_AFTER', 90),
        'after_commit' => true,
    ],
],
```

## When to migrate from `database` → `redis`

Watch for these signs:

- Worker tick latency rising (visible in `pg`/`mysql` slow query log)
- Lock waits on the `jobs` table
- Worker poll overhead approaching job execution time
- More than one worker server (the lock contention compounds)
- You start needing Horizon (which is Redis-only)

The migration is mechanical: provision Redis, change `QUEUE_CONNECTION`, restart workers. The `jobs` table is no longer used; archive or drop it after a transition period.

## Detection

```bash
# Current driver
grep 'QUEUE_CONNECTION' .env .env.example 2>/dev/null

# Spot the sync-in-prod foot-gun
grep 'QUEUE_CONNECTION=sync' .env

# Database queue at suspiciously high scale — count jobs/day from logs or DB:
# SELECT COUNT(*) FROM jobs WHERE created_at > NOW() - INTERVAL 1 DAY;
```

Reference: [Laravel 13 — Queues: Driver Notes and Prerequisites](https://laravel.com/docs/13.x/queues#driver-prerequisites)
