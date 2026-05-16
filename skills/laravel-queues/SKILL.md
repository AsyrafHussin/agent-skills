---
name: laravel-queues
description: Laravel 13 queue and job patterns — driver choice, job design (idempotency, ShouldQueue, model serialisation), retry and failure handling, worker scaling, Bus batching and chaining, Horizon when warranted, queue testing. Use when designing async jobs, scheduling background work, configuring Horizon, debugging stuck jobs, or auditing queue health. Triggers on "Laravel queue", "Laravel job", "background job", "Horizon setup", "failed jobs", "Bus batch", "queue worker tuning".
license: MIT
metadata:
  author: agent-skills
  version: "1.0.0"
---

# Laravel Queues & Jobs

Production-grade queue patterns for Laravel 13 (MySQL + Redis). Contains 20 rules across 6 categories covering driver choice, job design, retry/failure handling, worker scaling, batching/chaining, and testing. Targets the failure modes that pass code review but break under load — silent re-dispatch on retry, models stale on serialisation, missing idempotency on payment jobs, workers leaking memory, and "we forgot ShouldQueue and the request takes 12 seconds in production."

## Metadata

- **Version:** 1.0.0
- **Scope:** PHP / Laravel 13 + MySQL + Redis (queues), optional Horizon
- **Rule Count:** 20 rules across 6 categories
- **License:** MIT

## How to Use

When the user asks "design this background job", "audit our queue setup", "why is this job not running", or anything queue-related — work through this skill's rules as a checklist against the relevant files (job classes, `config/queue.php`, `config/horizon.php`, supervisor config, the dispatching call sites).

For audit mode, output per-rule verdicts:
- **PASS** — pattern correctly applied
- **FAIL** — anti-pattern present (with file:line + fix recommendation)
- **N/A** — does not apply to this codebase

End with a top-priority fix list (idempotency on payment jobs, missing `ShouldQueue`, supervisor `stopwaitsecs` too low — these are the most common production-bite issues).

## When to Apply

Reference this skill when:
- Writing a new background job (`php artisan make:job`)
- Reviewing a PR that adds or modifies a job class
- Setting up queue workers on a new server (Forge / Vapor / bare server)
- Configuring `config/queue.php` or `config/horizon.php`
- Debugging a stuck, looping, or repeatedly-failing job
- Choosing between `dispatch()`, `dispatchSync()`, `dispatchAfterResponse()`, `Bus::batch()`, `Bus::chain()`
- Adding queue testing (`Queue::fake()`, `Bus::fake()`)
- Deciding whether to adopt Horizon

## Step 1: Detect Queue Setup

Inspect:

| File | What to learn |
|---|---|
| `config/queue.php` | Default connection (`QUEUE_CONNECTION` env), failed-jobs storage, `after_commit` setting |
| `config/horizon.php` (if present) | Horizon environments, balance strategy, worker counts, timeouts |
| `app/Jobs/*.php` | Job classes, `ShouldQueue` usage, `tries`/`backoff`, `failed()` methods |
| `app/Console/Kernel.php` or `routes/console.php` | Scheduled jobs (`Schedule::job(...)`, `withoutOverlapping`) |
| `database/migrations/*_create_failed_jobs_table.php` | Failed-jobs storage migration; or DynamoDB driver in config |
| `supervisor*.conf` / `/etc/supervisor/conf.d/` | Worker process management, `numprocs`, `--max-time`, `stopwaitsecs` |

Typical setups:

| Stack | Queue driver | Failed driver | Worker manager |
|---|---|---|---|
| Small Laravel + MySQL | `database` | `database` | Supervisor (or systemd) |
| Production Laravel + Redis | `redis` | `database` (or `dynamodb` for serverless) | Supervisor + Horizon |
| Laravel Vapor | `sqs` | `dynamodb` | Vapor-managed |

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Driver & Config | CRITICAL | `config-` |
| 2 | Job Design | CRITICAL | `design-` |
| 3 | Retry & Failure | HIGH | `retry-` |
| 4 | Scaling & Workers | HIGH | `scaling-` |
| 5 | Batching & Chaining | HIGH | `bus-` |
| 6 | Testing & Operations | MEDIUM | `ops-` |

## Quick Reference

### 1. Driver & Config (CRITICAL)

- `config-driver-choice` — `database` for small apps; `redis` for production scale; `sqs` for Laravel Vapor
- `config-after-commit` — set `after_commit: true` to prevent dispatching jobs that reference uncommitted DB rows
- `config-failed-storage` — `failed_jobs` table is required (or DynamoDB on Vapor); set up retention/cleanup

### 2. Job Design (CRITICAL)

- `design-shouldqueue` — every async job MUST implement `ShouldQueue` (the #1 production bug: forgetting it makes the job run synchronously)
- `design-pass-ids-not-models` — pass IDs to the constructor; refetch in `handle()` — avoids stale models and bloated serialised payloads
- `design-idempotency` — payment, external API, and "create resource" jobs must be safe to run twice (`ShouldBeUnique`, idempotency keys, unique DB constraints)
- `design-constructor-vs-handle` — constructor runs at dispatch (sync); `handle()` runs on the worker. No DB writes or HTTP calls in the constructor.

### 3. Retry & Failure (HIGH)

- `retry-tries-and-backoff` — set both: `#[Backoff([1, 5, 30])]` for exponential delays; default 3 tries is usually too few for transient failures
- `retry-failed-method` — implement `failed(Throwable $e)` for permanent-failure handling (alert, refund, mark-as-failed)
- `retry-transient-vs-permanent` — `release($delay)` for transient errors (rate-limited, network); throw for permanent
- `retry-fail-on-timeout` — `#[FailOnTimeout]` to avoid burning all attempts on hung jobs

### 4. Scaling & Workers (HIGH)

- `scaling-supervisor-config` — Supervisor (or systemd) manages workers; `stopwaitsecs > timeout`; `--max-time=3600` to recycle
- `scaling-multi-queue-priority` — high/default/low queue lanes for SLA-critical jobs; `--queue=high,default,low`
- `scaling-worker-recycling` — `--max-jobs` and `--max-time` to combat memory leaks in long-running workers

### 5. Batching & Chaining (HIGH)

- `bus-batch-vs-chain` — `Bus::batch` for parallel + progress tracking; `Bus::chain` for strict sequential
- `bus-batch-failure-handling` — `allowFailures()` for fault-tolerant batches; use `then`/`catch`/`finally` callbacks
- `bus-chunking-large-sets` — for 1000+ items, chunk via `Bus::batch(...)` rather than one job per item

### 6. Testing & Operations (MEDIUM)

- `ops-queue-fake` — `Queue::fake()` / `Bus::fake()` in tests; `assertDispatched`, `assertPushed`, `assertChained`, `assertBatched`
- `ops-schedule-queued-jobs` — `Schedule::job(new X)->everyMinute()` queues the job; pair with `withoutOverlapping()` for safety
- `ops-horizon-when` — adopt Horizon when on Redis with multiple supervisors; not for `database` queue or single-worker setups

## Essential Patterns

### Minimum-viable job class (Laravel 13)

```php
<?php

namespace App\Jobs;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\Attributes\Backoff;
use Illuminate\Queue\Attributes\FailOnTimeout;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Throwable;

#[Backoff([1, 5, 30])]
#[FailOnTimeout]
class ChargeOrder implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public int $tries = 5;
    public int $timeout = 30;

    public function __construct(public readonly int $orderId) {}

    public function handle(StripeGateway $stripe): void
    {
        $order = Order::findOrFail($this->orderId);   // refetch — don't trust serialised state
        if ($order->status === 'paid') return;        // idempotency guard

        $charge = $stripe->charge($order);
        $order->markPaid($charge->id);
    }

    public function failed(Throwable $e): void
    {
        Order::find($this->orderId)?->markPaymentFailed($e->getMessage());
    }
}
```

### Dispatching

```php
ChargeOrder::dispatch($order->id);                       // default queue
ChargeOrder::dispatch($order->id)->onQueue('high');      // priority lane
ChargeOrder::dispatch($order->id)->delay(now()->addMinutes(5));
ChargeOrder::dispatchAfterResponse($order->id);          // run after HTTP response sent
```

### Supervisor config (recommended)

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/app/artisan queue:work redis --queue=high,default,low --sleep=3 --tries=3 --max-time=3600 --backoff=3
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/var/www/app/storage/logs/worker.log
stopwaitsecs=3600
```

**Critical:** `stopwaitsecs` must be greater than your longest job's timeout, or Supervisor will kill mid-job on deploy.

## References

- [Laravel 13 — Queues](https://laravel.com/docs/13.x/queues)
- [Laravel 13 — Horizon](https://laravel.com/docs/13.x/horizon)
- [Laravel 13 — Task Scheduling](https://laravel.com/docs/13.x/scheduling)
- [Supervisor — A Process Control System](http://supervisord.org/)

## Full Compiled Document

For the complete guide with all rules expanded: `AGENTS.md`
