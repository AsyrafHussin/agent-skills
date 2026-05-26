# Laravel Queues & Jobs - Complete Reference

**Version:** 1.0.0
**Organization:** Agent Skills Contributors
**Date:** May 2026
**License:** MIT

## Abstract

Production-grade queue patterns for **Laravel 13 (MySQL + Redis)**. Contains 20 rules across 6 categories covering driver choice, job design (idempotency, ShouldQueue, model serialisation), retry and failure handling, worker scaling and Supervisor config, Bus batching and chaining, queue testing, and when to adopt Horizon. Targets the failure modes that pass code review but break under load — silent re-dispatch on retry, models stale on serialisation, missing idempotency on payment jobs, workers leaking memory, and "we forgot ShouldQueue and the request takes 12 seconds in production."

## How to Audit

When asked to "audit our queue setup" or "review this job class", work through this skill's rules against the relevant files (job classes, `config/queue.php`, `config/horizon.php`, supervisor config, dispatch call sites). For each rule: **PASS** (correctly applied), **FAIL** (file:line + fix recommendation), or **N/A**.

## References

- [Laravel 13 — Queues](https://laravel.com/docs/13.x/queues)
- [Laravel 13 — Horizon](https://laravel.com/docs/13.x/horizon)
- [Laravel 13 — Task Scheduling](https://laravel.com/docs/13.x/scheduling)
- [Supervisor](http://supervisord.org/)

## Sections (definitions)


This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.


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

---


## after_commit: true — Wait for DB Transaction Before Dispatching

**Impact: CRITICAL (Without this, queued jobs reference rows that don't exist yet; jobs fail with ModelNotFoundException at random)**

When you dispatch a job inside a database transaction, the job can run on a worker **before the transaction commits**. The worker then tries to fetch the row by ID and gets nothing — `ModelNotFoundException`. The bug shows up intermittently in production and is maddening to debug.

`after_commit: true` tells Laravel to defer dispatch until the *outermost* transaction commits. Set it once in `config/queue.php` and you eliminate an entire class of race condition.

## The bug

```php
// ❌ Without after_commit — race condition

DB::transaction(function () use ($request) {
    $order = Order::create($request->validated());

    // Dispatched IMMEDIATELY — before the transaction commits.
    // The worker can pop and run this job in <1ms.
    ProcessOrder::dispatch($order->id);
});

// ProcessOrder::handle():
public function handle(): void
{
    $order = Order::findOrFail($this->orderId);   // ← may throw ModelNotFoundException
    // …
}
```

**What happens:**
1. Transaction begins
2. `Order::create(...)` runs inside the transaction (visible only within it)
3. `ProcessOrder::dispatch($order->id)` pushes to Redis with the new ID
4. Redis worker pops the job ~5ms later
5. Worker calls `findOrFail($orderId)` — the row isn't visible yet (still inside the controller's transaction)
6. `ModelNotFoundException` → job retries → may succeed by then, or burn all attempts

## Correct — configure once

```php
// ✅ config/queue.php
'connections' => [
    'redis' => [
        'driver' => 'redis',
        // …
        'after_commit' => true,   // ← every job on this connection waits for commit
    ],
    'database' => [
        'driver' => 'database',
        // …
        'after_commit' => true,
    ],
],
```

After this, the same controller code works correctly — the dispatch is queued until the transaction commits, then handed to the worker.

## Per-dispatch override

If you ever genuinely need to dispatch *before* commit (rare — usually a sign you should restructure), use:

```php
// ✅ Explicit override when you really want it
ProcessOrder::dispatch($order->id)->beforeCommit();
```

And the inverse, if `after_commit` is off globally but you want one safe dispatch:

```php
ProcessOrder::dispatch($order->id)->afterCommit();
```

## Why it matters even more with the `database` driver

The `database` queue driver writes the job row in the SAME database as your application — so it's caught in the same transaction. With `after_commit: false`, the job row is part of the rollback if the transaction fails — the dispatch silently disappears. `after_commit: true` ensures the job row is written *after* the commit, where it lives independently and is reliably picked up.

## Detection

```bash
# Check current config
grep -nE "'after_commit'" config/queue.php

# Find dispatches inside DB transactions that may rely on the row existing
grep -rEnB1 'dispatch\(' --include='*.php' app/Http/Controllers/ app/Actions/ app/Services/ \
  | grep -E 'DB::transaction|->transaction\(' | head
```

For new applications, **set `after_commit: true` from day one in every connection** in `config/queue.php`. It costs nothing and prevents a recurring class of bug.

Reference: [Laravel 13 — Queues: Jobs and Database Transactions](https://laravel.com/docs/13.x/queues#jobs-and-database-transactions)

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

---


## implements ShouldQueue — The #1 Production Bug

**Impact: CRITICAL (Without ShouldQueue, the job runs synchronously inside the request — what was meant as 'queued' blocks HTTP)**

The single most common Laravel queue bug: a class that uses `Dispatchable` and looks like a job, but doesn't implement `ShouldQueue`. Without that one interface marker, `dispatch()` calls `handle()` **inline**, blocking whatever called it. The 800ms "queued" Stripe charge isn't queued — it's adding 800ms to the checkout request.

AI-generated job scaffolding sometimes drops the `implements ShouldQueue` or names a "service" class as if it were a job. Always verify.

## Incorrect

```php
// ❌ No ShouldQueue — looks queued, runs sync

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class SendWelcomeEmail
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(public readonly int $userId) {}

    public function handle(): void
    {
        $user = User::findOrFail($this->userId);
        Mail::to($user)->send(new WelcomeEmail($user));   // takes 600ms
    }
}

// Controller:
public function register(Request $request)
{
    $user = User::create($request->validated());

    SendWelcomeEmail::dispatch($user->id);   // ← runs SYNCHRONOUSLY (no ShouldQueue!)

    return response()->json(['user' => $user]);
}
```

**What happens:**
- `dispatch()` checks: does this class implement `ShouldQueue`? **No.**
- Laravel: "OK, calling `handle()` immediately."
- 600ms email send happens inside the HTTP request
- User sees a 600ms+ register endpoint

**Why this happens:**
- Job class scaffolded but `implements ShouldQueue` forgotten
- Copy-pasted from a "Dispatchable" helper trait example
- AI generated a class that looks like a job but isn't

## Correct

```php
// ✅ Implements ShouldQueue — actually queues

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class SendWelcomeEmail implements ShouldQueue   // ← critical
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(public readonly int $userId) {}

    public function handle(): void { /* ... */ }
}

// Controller — same code, now actually async
SendWelcomeEmail::dispatch($user->id);
```

## Sanity check during a code review

When you see `Dispatchable` + `handle()`, immediately scan for `implements ShouldQueue`. If it's missing, that's the bug.

## The "dispatchSync intentionally" exception

Sometimes you genuinely want a job-style class to run sync — e.g., a complex orchestration you want to call from both controllers and queued contexts. Two acceptable patterns:

```php
// ✅ Always sync — don't pretend to be a job
final class CalculateInvoiceTotals  // no Dispatchable, no ShouldQueue
{
    public function __invoke(Invoice $invoice): Money { /* ... */ }
}
```

```php
// ✅ Sync OR queued, caller's choice — but the class IS a queued job
final class CalculateInvoiceTotals implements ShouldQueue { /* ... */ }

// To run sync from a controller:
CalculateInvoiceTotals::dispatchSync($invoice->id);
```

The bug is the *third* option — a class that *looks* like it queues but doesn't.

## Detection

```bash
# Find Dispatchable classes missing ShouldQueue
for f in app/Jobs/*.php app/Jobs/**/*.php; do
  if grep -q 'use Dispatchable' "$f" && ! grep -q 'implements ShouldQueue\|implements .* ShouldQueue' "$f"; then
    echo "MISSING ShouldQueue: $f"
  fi
done

# PHPStan rule (custom): require classes in app/Jobs/ to implement ShouldQueue
# (file a custom rule in your phpstan.neon ignoreErrors → become an architectural constraint)
```

Reference: [Laravel 13 — Queues: Creating Jobs](https://laravel.com/docs/13.x/queues#creating-jobs)

---


## Pass IDs to Jobs, Not Eloquent Models

**Impact: CRITICAL (Serialised models go stale; large payloads inflate the queue; SerializesModels refresh-from-DB has subtle traps)**

Constructor arguments to a queued job are serialised, written to the queue (Redis/MySQL), then deserialised on the worker. Passing an Eloquent model means:

- The entire serialised model (potentially with eager-loaded relations) lives in the queue payload
- `SerializesModels` trait refreshes the model from the DB on the worker — which sounds good, but means **the model state at dispatch is discarded**; any unsaved changes are lost
- If the model is deleted between dispatch and execution, the job throws `ModelNotFoundException`
- For collections, you ship the whole collection through Redis

**The fix is mechanical: pass IDs, refetch in `handle()`.**

## Incorrect

```php
// ❌ Pass the whole model — bloated payload, stale state

class SendOrderConfirmation implements ShouldQueue
{
    use Dispatchable, Queueable, SerializesModels;

    public function __construct(public readonly Order $order) {}

    public function handle(): void
    {
        // SerializesModels has refreshed $this->order from the DB.
        // Any unsaved state from the dispatcher is gone.
        Mail::to($this->order->user)->send(new OrderConfirmation($this->order));
    }
}

// Dispatch:
$order = Order::find(1)->load(['items', 'shipments', 'user.addresses']);
SendOrderConfirmation::dispatch($order);
// → Redis payload contains the order, its items, its shipments, its user, and the user's addresses.
// → Worker discards all of that and refetches `Order::find(1)` (without the relations).
// → 30KB payload for no benefit.
```

```php
// ❌ Pass a collection
class ProcessOrders implements ShouldQueue
{
    use Dispatchable, Queueable, SerializesModels;

    public function __construct(public readonly Collection $orders) {}
    // → Every order in the collection is serialised individually.
    // → A 1000-order batch dispatches a 1MB+ payload to Redis.
}
```

## Correct

```php
// ✅ Pass IDs; refetch in handle()

class SendOrderConfirmation implements ShouldQueue
{
    use Dispatchable, Queueable;

    public function __construct(public readonly int $orderId) {}

    public function handle(): void
    {
        $order = Order::with(['user', 'items'])->findOrFail($this->orderId);
        Mail::to($order->user)->send(new OrderConfirmation($order));
    }
}

// Dispatch:
SendOrderConfirmation::dispatch($order->id);    // 8-byte payload
```

```php
// ✅ Pass array of IDs for collection use cases
class ProcessOrders implements ShouldQueue
{
    use Dispatchable, Queueable;

    /** @param int[] $orderIds */
    public function __construct(public readonly array $orderIds) {}

    public function handle(): void
    {
        Order::whereIn('id', $this->orderIds)
            ->with('user')
            ->chunkById(100, fn ($chunk) => $chunk->each(fn ($o) => $this->processOne($o)));
    }
}
```

**Why it works:**
- Payload size is bounded (a list of integers)
- `handle()` decides exactly what to load (just `user`, not 5 eager-loaded relations)
- If the row was deleted between dispatch and execution, `findOrFail` throws — caller decides via `failed()` how to handle (often correct behaviour)
- Idempotency is easier — the ID is the natural key

## When passing the model IS acceptable

Two narrow cases:

1. **Tiny, immutable value objects** that don't have an `id` (e.g., a `Money` DTO, a config struct). These serialise cheaply and have no DB state to drift.
2. **Implicit Route Model Binding** in route definitions — not a queue concern.

If the value has a database id, **pass the id**.

## The `SerializesModels` trait clarified

`SerializesModels` doesn't "serialise" the model in the usual sense. At dispatch time, it stores the model's **class name + primary key**. At handle time, it issues a fresh `find($id)` (i.e., reads the current state from the DB). This is why:

- Stale state from the dispatcher is discarded — a feature for some cases, a footgun for others
- The model must still exist when the job runs (`ModelNotFoundException` if deleted)
- You can use `SerializesModels` AND pass the ID — but at that point, just pass the ID directly without the trait, and call `findOrFail` yourself in `handle()`

For new code: pass IDs, drop `SerializesModels` unless there's a specific need.

## Detection

```bash
# Find job constructors that accept Eloquent models or collections
grep -rEnB1 '__construct\(' --include='*.php' app/Jobs/ | \
  grep -E 'Model|Eloquent|Collection|EloquentBuilder|\\\\App\\\\Models\\\\'
```

Reference: [Laravel 13 — Queues: Class Structure](https://laravel.com/docs/13.x/queues#class-structure) · [Laravel 13 — Eloquent: Serialization](https://laravel.com/docs/13.x/eloquent-serialization)

---


## Idempotency — Safe to Re-run Without Duplicates

**Impact: CRITICAL (Without idempotency, retries duplicate payments, send double emails, and create phantom rows)**

A queued job CAN run more than once — not maybe, definitely. The worker process can crash mid-job (job released back to queue), the timeout can fire after the side effect succeeded (job tried again), Redis can reissue (rare but real). Every job that performs an external side effect (HTTP call to Stripe, email send, database insert with side effects) must be **idempotent**: safe to run N times with the same effect as 1.

## Three patterns for idempotency

### 1. State-check before action (cheapest)

```php
// ✅ Check before charging — if already paid, no-op

public function handle(StripeGateway $stripe): void
{
    $order = Order::findOrFail($this->orderId);

    if ($order->status === 'paid') {
        Log::info('Order already paid, skipping charge', ['order_id' => $order->id]);
        return;   // idempotent no-op
    }

    $charge = $stripe->charge($order);
    $order->markPaid($charge->id);
}
```

Works for jobs whose effect is reflected in a state field on the row.

### 2. Idempotency key sent to the upstream API

```php
// ✅ Stripe accepts Idempotency-Key headers — pass the same key for retries

public function handle(StripeGateway $stripe): void
{
    $order = Order::findOrFail($this->orderId);

    // Use a deterministic key per attempt — same value across retries of the same dispatch
    $idempotencyKey = sprintf('order:%d:charge', $order->id);

    $charge = $stripe->charges->create(
        [
            'amount'   => $order->total->cents(),
            'currency' => 'usd',
            'source'   => $order->payment_token,
        ],
        ['idempotency_key' => $idempotencyKey],     // ← critical
    );

    $order->update(['stripe_charge_id' => $charge->id, 'status' => 'paid']);
}
```

Stripe (and many APIs) deduplicate by idempotency key for 24 hours. Two retries of the same job → one real charge.

### 3. ShouldBeUnique — prevent dispatching duplicates in the first place

```php
// ✅ Reject dispatching while another instance is in flight

use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\UniqueFor;

#[UniqueFor(3600)]   // lock expires after 1 hour (in case worker dies)
class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    use Dispatchable, Queueable;

    public function __construct(public readonly int $productId) {}

    public function uniqueId(): string
    {
        return (string) $this->productId;
    }

    public function handle(): void { /* … */ }
}

// Now:
UpdateSearchIndex::dispatch(123);    // queues
UpdateSearchIndex::dispatch(123);    // silently discarded (another in flight with same uniqueId)
```

Use `ShouldBeUnique` for jobs where it would be a waste (not a bug) to enqueue duplicates — search-index rebuilds, cache warmers, etc.

## Incorrect — common bugs

```php
// ❌ No idempotency on a payment job
public function handle(StripeGateway $stripe): void
{
    $order = Order::findOrFail($this->orderId);
    $charge = $stripe->charge($order);              // ← runs again on retry → duplicate charge
    $order->update(['stripe_charge_id' => $charge->id]);
}

// ❌ Idempotency key derived from time (changes per retry → no dedup)
$idempotencyKey = sprintf('order:%d:%d', $order->id, time());

// ❌ "Mark paid" before charging — looks like idempotency, but a crash mid-step corrupts state
$order->update(['status' => 'paid']);
$charge = $stripe->charge($order);   // crash here = order marked paid, never charged
```

## A practical rule of thumb

Before merging a job, ask:

> "If this job ran twice with the same constructor arguments, would the second run cause:
> (a) no extra effect ✓
> (b) a duplicate side effect ✗
> (c) a different error ✗"

If the answer is (b) or (c), the job needs an idempotency mechanism.

## What about `attempts()` checks?

```php
// ⚠️ This is a SMELL — not idempotency, just avoidance

public function handle(): void
{
    if ($this->attempts() > 1) {
        Log::warning('Skipping on retry');
        return;
    }
    $stripe->charge($order);
}
```

This makes the job give up rather than safely retry. It's worse than no retry handling at all because it masks the symptom. Use real idempotency.

## Detection

```bash
# Jobs hitting external APIs without idempotency keys
grep -rEnB1 '->charges->create|->payments->create|Stripe::|Http::post\(' --include='*.php' app/Jobs/

# Then check each for either:
#  - state-check before action (early return)
#  - idempotency_key in the request payload
#  - ShouldBeUnique on the class
```

Reference: [Laravel 13 — Queues: Unique Jobs](https://laravel.com/docs/13.x/queues#unique-jobs) · [Stripe — Idempotent Requests](https://docs.stripe.com/api/idempotent_requests) · Internal: cross-cuts with `technical-debt`'s `defensive-missing-real` rule

---


## Constructor vs handle() — Keep the Constructor Pure

**Impact: HIGH (Constructor runs sync at dispatch; side effects there run inside the HTTP request)**

A common confusion: which code in a job class runs sync, and which runs on the worker?

| Method | Runs when | Runs where |
|---|---|---|
| **`__construct()`** | At `dispatch()` time | In the dispatching process (HTTP request) |
| `handle()` | When worker pops the job | In the queue worker process |
| `failed(Throwable $e)` | After all `$tries` exhausted | In the queue worker process |

Anything in `__construct()` happens **synchronously, inside the request that dispatches**. DB writes, HTTP calls, file I/O, slow computation — they all add to the request latency, defeating the point of queuing.

The constructor's job: **store primitive args for `handle()` to use.** Nothing else.

## Incorrect — DB writes in constructor

```php
// ❌ Constructor does work — adds latency to every dispatcher

class GenerateInvoicePdf implements ShouldQueue
{
    use Dispatchable, Queueable;

    public readonly string $pdfUrl;

    public function __construct(public readonly int $invoiceId)
    {
        // ⚠️ This runs in the HTTP request, not the worker.
        $invoice = Invoice::findOrFail($invoiceId);
        $renderer = app(InvoiceRenderer::class);
        $this->pdfUrl = $renderer->render($invoice);   // ← 1.2 seconds!

        // ⚠️ Database write in the constructor — caught by transactions, etc.
        $invoice->update(['pdf_generated_at' => now()]);
    }

    public function handle(): void
    {
        // The actual "queued" work is now a one-liner; the heavy lifting already happened.
        Mail::to($invoice->user)->send(new InvoiceReady($this->pdfUrl));
    }
}
```

**Why it breaks:**
- The dispatching controller now waits 1.2 seconds for the PDF render
- If the dispatcher is in a DB transaction, the constructor's `update()` is too — race conditions and rollback weirdness
- "Why is checkout slow?" — answered 3 days later: the constructor of an "async" job

## Correct — constructor stores args, handle() does the work

```php
// ✅ Constructor: pure assignment. handle(): the actual work.

class GenerateInvoicePdf implements ShouldQueue
{
    use Dispatchable, Queueable;

    public function __construct(public readonly int $invoiceId) {}

    public function handle(InvoiceRenderer $renderer): void
    {
        $invoice = Invoice::findOrFail($this->invoiceId);
        $pdfUrl = $renderer->render($invoice);
        $invoice->update(['pdf_generated_at' => now(), 'pdf_url' => $pdfUrl]);
        Mail::to($invoice->user)->send(new InvoiceReady($pdfUrl));
    }
}
```

**Why it works:**
- Dispatching takes microseconds (just enqueues the job ID)
- All the heavy work runs on the worker
- Dependencies (`InvoiceRenderer`) injected via `handle()` method signature — Laravel resolves from container at run time, not at dispatch

## What's OK in the constructor

- Assigning constructor arguments to properties (`public readonly` promotion in PHP 8+)
- Simple computed values from constructor arguments (e.g., calculating a string label from passed values)
- Setting `$this->onQueue('high')` for routing (rare; usually do at dispatch site instead)

What's NOT OK:
- `DB::*` queries
- `Http::*` calls
- `Mail::*`, `Storage::*`, anything that touches I/O
- `findOrFail` / Eloquent reads (defer to `handle()`)
- Heavy computation

## Dependency injection — `handle()` only

```php
// ✅ Inject services into handle(), not the constructor
public function handle(StripeGateway $stripe, OrderRepository $orders): void
{
    $order = $orders->findOrFail($this->orderId);
    $stripe->charge($order);
}
```

Laravel resolves `handle()` dependencies via the container at run time, on the worker. Injecting into the constructor doesn't work cleanly anyway — constructor args become part of the serialised payload, and you can't serialise a `StripeGateway` instance.

## Detection

```bash
# Find jobs whose constructor does more than property assignment
# (heuristic: constructor body > 3 lines is suspect)
for f in app/Jobs/*.php; do
  LINES=$(awk '/public function __construct/,/^[[:space:]]*\}/' "$f" | wc -l)
  if [ "$LINES" -gt 5 ]; then
    echo "SUSPECT (long constructor): $f"
  fi
done

# Find DB / HTTP / Mail calls inside __construct
grep -rEnB1 'public function __construct' --include='*.php' app/Jobs/ | \
  while read line; do
    f=$(echo "$line" | cut -d: -f1)
    awk '/public function __construct\(/,/^\s*\}/' "$f" | \
      grep -qE '(DB::|Http::|Mail::|Storage::|->find|->save|->update|->create)' && echo "SIDE EFFECT IN CTOR: $f"
  done
```

Reference: [Laravel 13 — Queues: Class Structure](https://laravel.com/docs/13.x/queues#class-structure) · [Laravel 13 — Queues: Dependency Injection](https://laravel.com/docs/13.x/queues#dependency-injection)

---


## $tries and #[Backoff] — Configure Both

**Impact: HIGH (Default 3 tries with no backoff causes retry storms; misconfigured backoff means slow recovery)**

Laravel's default retry behaviour is conservative: 1 attempt unless you set `$tries` (or the `--tries` flag on `queue:work`). That's almost never enough for jobs that touch external services — Stripe blips, network glitches, third-party rate limits all warrant a retry. But retrying immediately with no delay is just as bad — you hammer the same failing dependency 5 times in a row.

**Set both:** `$tries` (or `--tries` flag) and `#[Backoff(...)]` (or `$backoff` property) for delays. Use **exponential delays** for transient failures.

## Incorrect

```php
// ❌ Default 1 attempt — any transient failure is fatal
class ChargeCustomer implements ShouldQueue { /* … */ }
// No $tries → 1 attempt → first Stripe blip = job marked failed

// ❌ Many tries with no backoff — retry storm
class ChargeCustomer implements ShouldQueue
{
    public int $tries = 5;
    // No backoff → 5 immediate retries → if Stripe is rate-limiting you, you're rate-limited worse
}

// ❌ Constant backoff = naive
class ChargeCustomer implements ShouldQueue
{
    public int $tries = 5;
    public int $backoff = 30;
    // 5 retries × 30s apart = 150s recovery on a service that's been down for 5 minutes
}
```

## Correct — Laravel 13 idiom: `#[Backoff]` attribute with exponential delays

```php
// ✅ Production-grade retry config

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\Backoff;

#[Backoff([1, 5, 30, 120, 600])]   // exponential-ish: 1s, 5s, 30s, 2min, 10min
class ChargeCustomer implements ShouldQueue
{
    use Dispatchable, Queueable;

    public int $tries = 5;
    public int $timeout = 30;
    public int $maxExceptions = 3;   // stop retrying after 3 different exceptions

    // …
}
```

**What each setting does:**

| Setting | Effect |
|---|---|
| `$tries = 5` | Job will be released back to the queue up to 4 times before being marked failed |
| `#[Backoff([1, 5, 30, 120, 600])]` | Delay between retries: 1s, 5s, 30s, 2min, 10min |
| `$timeout = 30` | Each attempt has 30s before being killed (release or fail depending on `FailOnTimeout`) |
| `$maxExceptions = 3` | Even if `$tries` is 5, stop after 3 *distinct* exception types — protects against catastrophic bugs |

## Property vs attribute

Both forms work in Laravel 13. The attribute (`#[Backoff(...)]`) is preferred for new code — cleaner, sits next to the class declaration:

```php
// ✅ Attribute (preferred in Laravel 13)
#[Backoff([1, 5, 30])]
class X implements ShouldQueue {}

// ✅ Property (still works)
class X implements ShouldQueue {
    public int|array $backoff = [1, 5, 30];
}

// ✅ Computed backoff via method
public function backoff(): array
{
    return [1, 5, 30 * $this->attempts()];   // dynamic
}
```

## What value should `$tries` be?

| Job type | Recommended `$tries` |
|---|---|
| Payment / external charge | 5–7 (with idempotency) |
| External API call (Stripe, Shopify, etc.) | 3–5 |
| Email / notification | 3 |
| Internal pure computation | 1 (don't retry on real bugs) |
| Data export / batch processing | 3 (with chunking) |

For internal-only jobs (no external dependencies), `$tries = 1` is often right — retrying on a programmer error is just wasted compute.

## Setting at the worker level vs the job level

```bash
# Worker-level defaults (apply to all jobs unless overridden)
php artisan queue:work --tries=3 --backoff=3 --max-time=3600
```

Job-level settings (`$tries`, `#[Backoff]`) override worker-level. Use **job-level** for production — different jobs need different policies. Use **worker-level** as a sensible default for unconfigured jobs.

## Detection

```bash
# Jobs without $tries or #[Tries] / #[Backoff]
for f in app/Jobs/*.php; do
  if grep -q 'implements ShouldQueue' "$f" && ! grep -qE '\$tries|#\[Tries|#\[Backoff' "$f"; then
    echo "NO RETRY CONFIG: $f"
  fi
done

# Find jobs with constant backoff (= 0 or a single int) on external-facing work
grep -rEn 'public int \$backoff = [0-9]+;' --include='*.php' app/Jobs/
```

Reference: [Laravel 13 — Queues: Max Job Attempts & Timeout](https://laravel.com/docs/13.x/queues#max-job-attempts-and-timeout) · [Laravel 13 — Queues: Backoff Configuration](https://laravel.com/docs/13.x/queues#max-job-attempts-and-timeout)

---


## failed(Throwable $e) — Handle Permanent Failures

**Impact: HIGH (Without failed(), permanent failures are silent; customers see orders stuck in 'pending')**

When a job exhausts all `$tries` (or hits `$maxExceptions`), Laravel writes the job to `failed_jobs` and stops. If you don't implement `failed(Throwable $e)`, that's the end of it — the side effect is incomplete, the row in your DB is in a half-state, the customer never finds out.

`failed()` is your hook for **terminal failure handling**: revert state, notify, refund, alert. Implement it for every job whose terminal failure has a meaningful response.

## Incorrect — no `failed()` handler

```php
// ❌ Job permanently fails; the order is left in 'pending' status forever

class ChargeOrder implements ShouldQueue
{
    use Dispatchable, Queueable;

    public int $tries = 5;

    public function __construct(public readonly int $orderId) {}

    public function handle(StripeGateway $stripe): void
    {
        $order = Order::findOrFail($this->orderId);
        if ($order->status === 'paid') return;

        $charge = $stripe->charge($order);
        $order->markPaid($charge->id);
    }
    // No failed() method!
}

// What happens:
// 1. Stripe API down for 1 hour
// 2. All 5 attempts fail
// 3. Job written to failed_jobs
// 4. Order stays in 'pending' status indefinitely
// 5. Customer emails support 2 days later: "you charged me, why is my order still pending?"
//    (Actually you DIDN'T charge them — but no one knows because nothing surfaced)
```

## Correct — implement `failed()`

```php
// ✅ Terminal-failure handler reverts state and surfaces the problem

class ChargeOrder implements ShouldQueue
{
    use Dispatchable, Queueable;

    public int $tries = 5;

    public function __construct(public readonly int $orderId) {}

    public function handle(StripeGateway $stripe): void { /* … as before */ }

    public function failed(Throwable $e): void
    {
        $order = Order::find($this->orderId);
        if (!$order) return;   // order deleted; nothing to do

        $order->markPaymentFailed($e->getMessage());

        // Surface to customer support / on-call
        report($e);   // → Sentry / Bugsnag

        // Notify customer of the problem (with a retry link or contact info)
        Mail::to($order->user)->send(new PaymentFailedNotification($order, $e));

        // Slack / paging — payment failures matter
        Notification::route('slack', config('app.alerts_webhook'))
            ->notify(new OrderPaymentFailed($order));
    }
}
```

**What `failed()` typically does:**

1. **Revert / update state** — mark the row as failed so downstream queries see the correct state
2. **Report the exception** — `report($e)` sends to Sentry / Bugsnag / Rollbar
3. **Notify users** — for user-facing failures (payment, signup), email or in-app notification
4. **Alert the team** — for high-stakes failures, Slack / PagerDuty / OpsGenie
5. **Optionally re-dispatch later** — for retries beyond the queue's policy (e.g., "try again tomorrow")

## What `failed()` should NOT do

- **Retry the same job** — that's what the queue's retry mechanism is for. If you need to retry differently, dispatch a *different* job (`RetryChargeOrder::dispatch(...)` next day).
- **Throw exceptions** — `failed()` shouldn't fail. Wrap in try/catch if any of its operations could throw.

## Per-job vs global failure listener

You can also wire a global `JobFailed` event listener (covered in [config-failed-storage](config-failed-storage.md)) for cross-cutting concerns (Sentry reporting, metrics). The per-job `failed()` is for **job-specific** semantics (order-specific state revert).

Best practice: **both**.

- Global listener → reports to Sentry, increments a Prometheus counter, sends to Slack
- Per-job `failed()` → job-specific side-effect reversal (mark order as failed, refund the hold, etc.)

## Detection

```bash
# Jobs whose handle() touches external services but have no failed() method
for f in app/Jobs/*.php; do
  has_external=$(grep -qE 'Http::|Mail::|Stripe::|Notification::' "$f" && echo y || echo n)
  has_failed=$(grep -qE 'public function failed\b' "$f" && echo y || echo n)
  if [ "$has_external" = "y" ] && [ "$has_failed" = "n" ]; then
    echo "MISSING failed(): $f"
  fi
done
```

Reference: [Laravel 13 — Queues: Cleaning Up After Failed Jobs](https://laravel.com/docs/13.x/queues#cleaning-up-after-failed-jobs) · [Laravel 13 — Queues: Job Failed Event Listeners](https://laravel.com/docs/13.x/queues#failed-job-events)

---


## Transient vs Permanent Failures — release() or throw?

**Impact: HIGH (Treating every error as retryable wastes attempts; treating none as retryable fails on transient blips)**

Not all failures deserve a retry. A failed `findOrFail` (the row was deleted) won't be fixed by retrying. A 402 Payment Required from Stripe means the card was declined — retrying with the same token won't help. But a 503 from Stripe's status page is transient — retry IS the right answer.

The job's `handle()` method should distinguish:

- **Transient** — release with delay (`$this->release(60)`) so the job sits in the queue and tries again later
- **Permanent** — throw (let the job hit `failed()`); future retries can't help
- **Rate-limited** — release with the Retry-After value if the upstream told you when to come back

## The three responses

```php
public function handle(StripeGateway $stripe): void
{
    $order = Order::findOrFail($this->orderId);
    if ($order->status === 'paid') return;

    try {
        $charge = $stripe->charge($order);
    } catch (CardException $e) {
        // ✅ PERMANENT — card declined, no retry will help
        $order->markCardDeclined($e->getStripeCode());
        throw $e;                       // → failed_jobs after one attempt
    } catch (RateLimitException $e) {
        // ✅ RATE LIMITED — release with the delay Stripe asked for
        $this->release($e->retryAfterSeconds());
        return;                         // job stays in queue, doesn't count against $tries (in Redis driver)
    } catch (ConnectException $e) {
        // ✅ TRANSIENT — network error, retry with backoff
        throw $e;                       // → counts as an attempt; #[Backoff] kicks in
    }

    $order->markPaid($charge->id);
}
```

## `release()` vs `throw` — what's the difference?

| Action | Effect on `$tries` count | When to use |
|---|---|---|
| `$this->release($seconds)` | Does NOT count as an attempt; job stays in queue with delay | Rate-limited; you know it'll work later; want to avoid burning attempts |
| `throw $e` | Counts as an attempt; `#[Backoff]` applies between this attempt and next | Transient error of unknown duration; want exponential backoff |
| `$this->fail($e)` | Job moves straight to `failed_jobs`, `failed()` called | Permanent failure; don't waste more attempts |

**Quick mental model:**
- **You know how long to wait** → `release($seconds)`
- **You don't know how long** → `throw`
- **It will never succeed** → `fail($e)` or `throw` a domain exception that you understand to be permanent in `failed()`

## Incorrect

```php
// ❌ Treats everything the same — every error retries

public function handle(): void
{
    $order = Order::findOrFail($this->orderId);     // ModelNotFoundException is retried 5 times
    $charge = $stripe->charge($order);              // CardException is retried 5 times
    // ...
}
```

```php
// ❌ Manual retry loop inside handle() — fighting the queue

public function handle(): void
{
    for ($i = 0; $i < 5; $i++) {
        try {
            $charge = $stripe->charge($order);
            break;
        } catch (Exception $e) {
            sleep(2);   // ← blocks the worker doing nothing useful
        }
    }
}
```

```php
// ❌ Catch + log + return — swallowing failures

public function handle(): void
{
    try {
        $charge = $stripe->charge($order);
    } catch (Exception $e) {
        Log::error('Charge failed');
        return;       // job "succeeded" silently
    }
}
```

## Distinguishing in practice

For Stripe (typical):
- `Stripe\Exception\CardException` (4xx) → **permanent** (card declined, expired, etc.)
- `Stripe\Exception\InvalidRequestException` (4xx) → **permanent bug** (your code is wrong)
- `Stripe\Exception\AuthenticationException` (401) → **permanent infra** (wrong key)
- `Stripe\Exception\RateLimitException` (429) → **rate-limited**, release with delay
- `Stripe\Exception\ApiConnectionException` → **transient**, throw to retry
- `Stripe\Exception\ApiErrorException` (5xx) → **transient**, throw to retry

For HTTP via `Http::*`:
- `Http::failed()` (4xx/5xx) → check the status; 5xx is transient, 4xx usually permanent

For database:
- `QueryException` with deadlock / lock-wait → **transient**, throw
- `QueryException` with constraint violation → **permanent**, throw (and check your dispatch site)
- `ModelNotFoundException` → **permanent** (row gone; nothing to retry)

## Detection

```bash
# Jobs that catch \Exception (or \Throwable) without distinguishing
grep -rEnA3 'catch\s*\(\s*\\?Exception\b' --include='*.php' app/Jobs/

# Jobs using release() — verify they're using it for rate-limit / known-wait scenarios
grep -rEnB1 'this->release\(' --include='*.php' app/Jobs/

# Jobs with sleep() — almost always wrong (blocks worker)
grep -rEnH '\bsleep\(' --include='*.php' app/Jobs/
```

Reference: [Laravel 13 — Queues: Releasing a Job](https://laravel.com/docs/13.x/queues#error-handling) · [Laravel 13 — Queues: Manually Failing a Job](https://laravel.com/docs/13.x/queues#error-handling)

---


## #[FailOnTimeout] — Don't Burn Attempts on Hangs

**Impact: HIGH (A hung HTTP call kills 5 attempts at 30s each before failing — 2.5 min of wasted worker time per stuck job)**

Without `#[FailOnTimeout]`, a job that exceeds `$timeout` is **released back to the queue** and tries again — the same way as a job that threw an exception. For a hung HTTP call (third-party API not responding), the second attempt will likely hang too. And the third. With `$tries = 5` and `$timeout = 30`, a single stuck dispatch consumes 150 seconds of worker time across 5 attempts.

`#[FailOnTimeout]` tells Laravel: "if this job times out, it's done. Mark it failed; don't retry." That's almost always what you want for timeout-prone jobs.

## Incorrect — default behaviour wastes attempts

```php
// ❌ Without FailOnTimeout, timeouts are released and retried

#[Backoff([1, 5, 30])]
class FetchOrderFromShopify implements ShouldQueue
{
    use Dispatchable, Queueable;

    public int $tries = 5;
    public int $timeout = 30;

    public function handle(): void
    {
        // Shopify is down; this HTTP call hangs for the full 30s timeout
        $data = Http::get("https://api.shopify.com/orders/{$this->orderId}")->json();
        // …
    }
}
```

**Timeline of a hung job:**

| Attempt | Outcome | Wait before next | Time elapsed |
|---|---|---|---|
| 1 | Times out after 30s | 1s backoff | 31s |
| 2 | Times out after 30s | 5s backoff | 66s |
| 3 | Times out after 30s | 30s backoff | 126s |
| 4 | Times out after 30s | (no more) | 156s |
| 5 | Times out after 30s | — | 186s |

3 minutes of worker time burned. Meanwhile real jobs are waiting behind it.

## Correct — `#[FailOnTimeout]` for I/O-bound jobs

```php
// ✅ Timeout = immediate failure; don't burn more attempts

use Illuminate\Queue\Attributes\Backoff;
use Illuminate\Queue\Attributes\FailOnTimeout;

#[Backoff([1, 5, 30])]
#[FailOnTimeout]
class FetchOrderFromShopify implements ShouldQueue
{
    use Dispatchable, Queueable;

    public int $tries = 5;
    public int $timeout = 30;

    public function handle(): void
    {
        // Shopify hangs → job dies at 30s → failed() called → no further retries
        $data = Http::timeout($this->timeout - 1)         // belt-and-braces: HTTP-level timeout too
            ->get("https://api.shopify.com/orders/{$this->orderId}")
            ->json();
        // …
    }

    public function failed(Throwable $e): void
    {
        report($e);   // alert; investigate Shopify status
    }
}
```

Now the same hung job burns 30 seconds, hits `failed()`, and the worker moves on. The other jobs queued behind don't suffer.

## When NOT to use `#[FailOnTimeout]`

For jobs where the timeout is set generously and a single timeout might just mean "the worker was busy" rather than "the operation is genuinely stuck":

- Long-running data processing (PDF generation, video transcoding)
- Operations that get faster on subsequent retries (caches warm up, etc.)
- Jobs where the timeout is a generous upper bound rather than a strict deadline

For those, **let the timeout release the job for retry.** But pair with a sensible `$tries` (3 is fine) and `#[Backoff]` so you don't loop tightly.

## Setting the HTTP-level timeout to less than the job timeout

A common bug: the job's `$timeout = 30` but the HTTP client has no timeout, so the call hangs longer than the job, the job killer interrupts mid-write, and you get partial state. Always:

```php
$response = Http::timeout($this->timeout - 1)   // shave 1s for cleanup
    ->retry(0)                                   // disable client-level retry; let the job's retry handle it
    ->post('https://api.example.com/charge', $payload);
```

Cross-references the [`technical-debt`'s `defensive-missing-real`](../../technical-debt/rules/defensive-missing-real.md) rule.

## Detection

```bash
# Jobs that touch external services without #[FailOnTimeout]
for f in app/Jobs/*.php; do
  has_io=$(grep -qE 'Http::|Stripe::|Mail::' "$f" && echo y || echo n)
  has_fail_on_timeout=$(grep -qE '#\[FailOnTimeout\]' "$f" && echo y || echo n)
  has_timeout=$(grep -qE '\$timeout\s*=' "$f" && echo y || echo n)

  if [ "$has_io" = "y" ] && [ "$has_fail_on_timeout" = "n" ]; then
    echo "CONSIDER #[FailOnTimeout]: $f (touches I/O, no FailOnTimeout)"
  fi
done

# Jobs with no $timeout at all (uses worker default of 60s)
for f in app/Jobs/*.php; do
  grep -q 'implements ShouldQueue' "$f" || continue
  grep -qE '\$timeout\s*=|--timeout' "$f" || echo "NO TIMEOUT SET: $f"
done
```

Reference: [Laravel 13 — Queues: Failing on Timeout](https://laravel.com/docs/13.x/queues#job-expirations-and-timeouts)

---


## Supervisor Config — Don't Kill Mid-Job on Deploy

**Impact: HIGH (Wrong stopwaitsecs cuts jobs mid-execution on every deploy — corrupted state, duplicate charges)**

Laravel queue workers run forever. They need a process manager to start them, restart them if they crash, and stop them gracefully on deploy. **Supervisor** is the standard (also valid: systemd, Procfile on Heroku, Forge's UI which wraps Supervisor).

The most common production bug: **`stopwaitsecs` set too low.** When Supervisor sends SIGTERM on `php artisan queue:restart`, the worker finishes its current job before exiting. If Supervisor kills the worker (SIGKILL) before the job finishes, you get partial state — half-charged customer, half-imported CSV, exception logged but row not updated.

## Anatomy of a correct Supervisor config

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

**What each directive does:**

| Directive | Meaning | Critical to get right? |
|---|---|---|
| `command` | The worker invocation | **Yes** — see flags below |
| `numprocs` | Number of worker processes (parallel job slots) | Yes — undersized = backed-up queues |
| `autorestart=true` | If a worker crashes, restart it | Yes |
| `stopasgroup=true` + `killasgroup=true` | Stop all child processes too | Yes |
| `stopwaitsecs` | How long to wait for graceful shutdown before SIGKILL | **Critical** — must exceed your longest job's `$timeout` |
| `redirect_stderr` + `stdout_logfile` | Capture worker output | Useful for debugging |

## The `stopwaitsecs` trap

A job has `$timeout = 30`. The worker is running it. Deploy triggers `supervisorctl restart laravel-worker`. Supervisor sends SIGTERM:

- **`stopwaitsecs = 10`** ❌ — Supervisor waits 10s, then SIGKILL. Job is mid-charge. Customer charged, your DB not updated. Manual reconciliation tomorrow.
- **`stopwaitsecs = 3600`** ✅ — Supervisor waits up to an hour. Worker finishes its current 30s job, exits cleanly. Deploy proceeds.

**Rule:** `stopwaitsecs > max(jobs' $timeout values)` with healthy margin. **Default to 3600 (1 hour)** unless you have a specific reason otherwise. Idle workers exit immediately on SIGTERM; only busy ones wait.

## Worker command flags

```bash
php artisan queue:work redis \
    --queue=high,default,low \      # priority lanes (see scaling-multi-queue-priority)
    --sleep=3 \                     # how long to sleep when no jobs available
    --tries=3 \                     # default $tries unless job overrides
    --max-time=3600 \               # restart worker after 1h (combat memory leaks)
    --backoff=3                     # default backoff unless job overrides
```

The two most important flags for production:

- **`--max-time=3600`** — recycles the worker process every hour. PHP isn't great at long-running processes; recycling combats memory drift.
- **`--queue=high,default,low`** — pull from `high` first, then `default`, then `low`. Without this you have no priority lanes.

## Systemd alternative (modern servers)

If you don't want Supervisor, systemd works:

```ini
# /etc/systemd/system/laravel-worker@.service
[Unit]
Description=Laravel queue worker %i
After=redis.service mysql.service

[Service]
User=forge
Group=forge
Restart=always
RestartSec=5
ExecStart=/usr/bin/php /var/www/app/artisan queue:work redis --queue=high,default,low --sleep=3 --tries=3 --max-time=3600 --backoff=3
TimeoutStopSec=3600
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

```bash
# Enable 8 workers
systemctl enable laravel-worker@{1..8}.service
systemctl start laravel-worker@{1..8}.service
```

Equivalent semantically; pick based on team familiarity.

## Avoiding the "jobs not running" mystery

Symptoms: dispatched jobs appear in `jobs` table / Redis but never run. Diagnosis order:

1. **Is the worker running?** `supervisorctl status laravel-worker:*`
2. **Is it pulling the right queue?** Check `--queue=` flag matches your `onQueue(...)` dispatches
3. **Is it pulling the right connection?** Check first arg to `queue:work` (e.g., `redis` vs `database`)
4. **Did you forget `queue:restart` after a deploy?** Workers boot the old code until restarted
5. **Did the worker crash?** Check `worker.log` and Supervisor's `autostartretries`

## Detection

```bash
# Find supervisor config files
find /etc/supervisor /etc/supervisord.d ~/supervisor 2>/dev/null -name '*laravel*' -o -name '*queue*'

# Validate stopwaitsecs is reasonable
grep -nH 'stopwaitsecs' /etc/supervisor/conf.d/*.conf 2>/dev/null \
  | awk -F= '{ if ($2 < 60) print "TOO LOW: " $0 }'

# On Forge: the worker config is in the panel under "Daemons"; verify Time Out is high enough.
```

Reference: [Laravel 13 — Queues: Supervisor Configuration](https://laravel.com/docs/13.x/queues#supervisor-configuration) · [Supervisor docs](http://supervisord.org/)

---


## Multi-Queue Priority Lanes — high/default/low

**Impact: HIGH (Without priority lanes, a flood of low-priority emails stalls SLA-critical payment jobs)**

By default, every Laravel job dispatches to the `default` queue. Every worker pulls from `default`. When a batch of 50,000 marketing emails hits at the same time as a payment-charge job, the payment waits behind the marketing emails.

**Priority lanes** fix this. Define multiple queues (`high`, `default`, `low`), tell each dispatch where to go, tell workers to drain in priority order. Payment charges land in `high`; marketing emails land in `low`; workers pull `high` first.

## The setup

### 1. Tell workers to drain in priority order

```bash
# Supervisor command
php artisan queue:work redis --queue=high,default,low --sleep=3 --tries=3 --max-time=3600
```

The `--queue=high,default,low` flag tells the worker: "always check `high` first; if empty, check `default`; if empty, check `low`." A single worker, multiple queues, strict priority.

### 2. Dispatch each job to the correct queue

```php
// SLA-critical: payments, password resets, real-time notifications
ChargeOrder::dispatch($order->id)->onQueue('high');

// Default: most user-facing async work
SendOrderConfirmation::dispatch($order->id);   // → 'default'

// Low-priority: bulk marketing, analytics, periodic recalculation
SendMarketingDigest::dispatch($user->id)->onQueue('low');
```

Or as a class default on the job:

```php
class ChargeOrder implements ShouldQueue
{
    use Dispatchable, Queueable;

    public function __construct(public readonly int $orderId)
    {
        $this->onQueue('high');
    }
}
```

### 3. (Optional but recommended) Reserve dedicated workers for high-priority

If `high` is critical, run a dedicated set of workers that ONLY drain `high`:

```bash
# Supervisor program 1: high-priority workers (4 of them, always available for SLA work)
php artisan queue:work redis --queue=high --sleep=3 --tries=3 --max-time=3600
numprocs=4

# Supervisor program 2: general workers
php artisan queue:work redis --queue=high,default,low --sleep=3 --tries=3 --max-time=3600
numprocs=8
```

Why: even with priority order, a slow `low` job already in-flight on a worker delays the next `high` job that worker picks up. Dedicated `high`-only workers eliminate this.

## Incorrect

```php
// ❌ Everything on 'default' — no SLA differentiation

ChargeOrder::dispatch($order->id);             // default
SendMarketingDigest::dispatch($user->id);      // default
SendOrderConfirmation::dispatch($order->id);   // default
```

```bash
# ❌ Worker not pulling from priority queues
php artisan queue:work --queue=default

# All those marketing emails sitting in 'default' — they pile up alongside payment jobs.
```

## Queue naming — keep it simple

Don't over-engineer. The dominant pattern is **three lanes**:

| Lane | What goes here | Typical SLA |
|---|---|---|
| **`high`** | Payment, auth (password reset, MFA), real-time user-visible | < 5 seconds |
| **`default`** | Most async work (emails, notifications, exports) | < 30 seconds |
| **`low`** | Bulk operations (marketing emails, batch reports, periodic recalculation) | "When the queue is idle" |

Resist the urge to add `medium`, `urgent`, `payments-only`, `notifications`, `emails-marketing`, `emails-transactional`, `analytics`, etc. — five lanes is the most teams maintain without losing track. Three lanes is plenty.

## Horizon equivalent

If using Horizon (Redis only), the same three lanes are configured in `config/horizon.php`:

```php
'environments' => [
    'production' => [
        'supervisor-high' => [
            'queue' => ['high'],
            'balance' => 'simple',
            'processes' => 4,
            'timeout' => 60,
        ],
        'supervisor-default' => [
            'queue' => ['high', 'default', 'low'],
            'balance' => 'auto',
            'maxProcesses' => 12,
            'minProcesses' => 4,
            'timeout' => 60,
        ],
    ],
],
```

Horizon's `auto` balancing dynamically allocates processes between queues based on load — useful when load varies. See [`ops-horizon-when`](ops-horizon-when.md) for when adopting Horizon makes sense.

## Detection

```bash
# Are there any onQueue() calls in the codebase?
grep -rEn "->onQueue\(" --include='*.php' app/ | head

# What queues are workers actually pulling from?
grep -nH '\--queue' /etc/supervisor/conf.d/*.conf supervisor*.conf 2>/dev/null

# Verify workers drain in priority order (high → default → low)
ps aux | grep 'queue:work'
```

If no `--queue=` flag, the worker pulls only `default`. If `onQueue()` calls exist but the worker doesn't list those queues, those jobs never run.

Reference: [Laravel 13 — Queues: Dispatching to a Particular Queue](https://laravel.com/docs/13.x/queues#customizing-the-queue-and-connection) · [Laravel 13 — Horizon: Balancing Strategy](https://laravel.com/docs/13.x/horizon#balancing-strategies)

---


## Worker Recycling — --max-jobs and --max-time

**Impact: MEDIUM (Long-running PHP processes leak memory; recycling combats drift before it bites)**

PHP wasn't designed for forever-running processes. Workers accumulate memory over time — Eloquent's IdentityMap, container-bound singletons, Carbon objects in static caches, third-party library state. After a few thousand jobs or a few hours, a worker can be using 4× the RAM it started with.

Laravel's queue worker has two flags that solve this **by gracefully exiting and letting Supervisor restart the process**:

- **`--max-jobs=N`** — exit after processing N jobs
- **`--max-time=N`** — exit after N seconds

The worker finishes its current job, exits, and Supervisor immediately restarts a fresh process. Memory resets to baseline.

## The default (without these flags)

```bash
# ❌ Worker runs forever — memory grows
php artisan queue:work redis
```

Symptoms:
- `htop` shows worker processes at 800MB-1.5GB after a day
- OOM-killed by the kernel on memory-constrained servers
- Worker performance degrades — late-life job latency 2-3× higher than fresh-process latency
- Occasional `Allowed memory size exhausted` in failed jobs

## Correct

```bash
# ✅ Recycle every hour (most common)
php artisan queue:work redis --sleep=3 --tries=3 --max-time=3600

# ✅ Or recycle every 1000 jobs (when traffic is bursty)
php artisan queue:work redis --sleep=3 --tries=3 --max-jobs=1000

# ✅ Both (whichever hits first)
php artisan queue:work redis --sleep=3 --tries=3 --max-time=3600 --max-jobs=1000
```

When the limit hits, the worker exits cleanly between jobs (it doesn't interrupt the current one). Supervisor then restarts it.

## Why `--memory` isn't enough

Laravel also has `--memory=128`, which kills the worker if memory usage exceeds 128MB. This works but:

- It triggers AFTER the leak has grown to the limit (degraded performance period)
- It can cut a job mid-execution if the job itself spikes memory
- It depends on accurate memory reporting (sometimes off on container platforms)

`--max-time` and `--max-jobs` are **preventive**; `--memory` is **emergency brake**. Use the preventive flags by default; use `--memory` as a belt-and-braces upper bound.

```bash
# All three
php artisan queue:work redis --sleep=3 --tries=3 --max-time=3600 --max-jobs=1000 --memory=256
```

## Picking the right values

| Worker type | `--max-time` | `--max-jobs` |
|---|---|---|
| Default | 3600 (1h) | 1000 |
| Memory-heavy jobs (PDF, video) | 600 (10min) | 50 |
| Lightweight jobs (notifications, cache warmers) | 7200 (2h) | 5000 |
| Burst processing (data import) | 1800 (30min) | 500 |

These are starting points. Watch your worker memory over time and adjust.

## What happens during recycle

1. Worker finishes its current job (or exits during the idle sleep if no job)
2. Worker process exits cleanly (zero exit code)
3. Supervisor (or systemd) immediately starts a new worker
4. Cold-start cost: ~200-500ms while the new worker boots Laravel
5. Net effect: a few hundred ms of "no available worker" per recycle, far less than the memory-leak cost

For an 8-worker pool with `--max-time=3600`, you get roughly one cold-start every 7.5 minutes across the fleet — negligible.

## On Horizon

Horizon manages worker lifecycle differently — workers exit when Horizon decides via the configured `nice` and `tries` settings. The equivalent recycling happens via Horizon's auto-balancer and restart on deploy. You typically don't set `--max-time` on Horizon-managed workers; let Horizon handle it.

## Detection

```bash
# Workers running without recycling flags
grep -nH 'queue:work' /etc/supervisor/conf.d/*.conf 2>/dev/null \
  | grep -v -- '--max-time\|--max-jobs'

# Current worker memory usage (sanity check)
ps aux | grep 'queue:work' | awk '{ printf "%s\t%.1f MB\n", $11, $6/1024 }'
```

If any active worker is using > 500MB, recycling is overdue.

Reference: [Laravel 13 — Queues: Processing a Specified Number of Jobs](https://laravel.com/docs/13.x/queues#the-queue-work-command) · [Laravel 13 — Queues: Resource Considerations](https://laravel.com/docs/13.x/queues#queue-workers-and-deployment)

---


## Bus::batch vs Bus::chain — Pick the Right One

**Impact: HIGH (Wrong choice means parallel work runs serial (slow) or sequential work runs in parallel (race conditions))**

When dispatching multiple related jobs, Laravel offers two APIs that look similar but behave very differently:

- **`Bus::chain([...])`** — strictly sequential. Each job runs only after the previous one succeeds. If any job fails, the rest are abandoned.
- **`Bus::batch([...])`** — parallel by default. All jobs queue immediately; workers pick them up concurrently. Tracks aggregate progress; offers `then`, `catch`, `finally` callbacks.

Picking the wrong one creates real bugs.

## Decision tree

| Scenario | Use |
|---|---|
| Step B depends on step A's side effects | `Bus::chain` (sequential, abort on failure) |
| Process 1000 items, order doesn't matter | `Bus::batch` (parallel) |
| Want a "done when all are done" callback | `Bus::batch` |
| Want a progress bar / status page | `Bus::batch` |
| Workflow with branching ("do A, then B and C in parallel, then D") | `Bus::chain` containing `Bus::batch` |
| One job's failure should NOT stop the others | `Bus::batch->allowFailures()` |
| One job's failure SHOULD stop everything after | `Bus::chain` |

## Bus::chain — strict sequence

```php
// ✅ Each step depends on the previous
use App\Jobs\OptimizePodcast;
use App\Jobs\ProcessPodcast;
use App\Jobs\ReleasePodcast;
use Illuminate\Support\Facades\Bus;

Bus::chain([
    new ProcessPodcast(podcastId: 42),     // step 1
    new OptimizePodcast(podcastId: 42),    // step 2 — runs only if step 1 succeeded
    new ReleasePodcast(podcastId: 42),     // step 3 — runs only if step 2 succeeded
])->dispatch();
```

If `ProcessPodcast` fails (all tries exhausted), `OptimizePodcast` and `ReleasePodcast` are never dispatched. The chain stops dead.

You can add `catch` to handle chain failure:

```php
Bus::chain([
    new ProcessPodcast(42),
    new OptimizePodcast(42),
    new ReleasePodcast(42),
])->catch(function (Throwable $e) {
    // Chain failed; clean up partial state
    Podcast::find(42)?->markProcessingFailed($e->getMessage());
})->dispatch();
```

## Bus::batch — parallel + aggregate tracking

```php
// ✅ Process 500 CSV rows in parallel, with progress callbacks
use App\Jobs\ImportCsvRow;
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\Bus;
use Throwable;

$batch = Bus::batch(
    Csv::rows($file)->map(fn ($row) => new ImportCsvRow($row))
)->name('CSV import — ' . $file->name)
 ->before(function (Batch $batch) {
    // Batch has been created, jobs not yet added
 })
 ->progress(function (Batch $batch) {
    // Called after each job completes
    broadcast(new BatchProgressUpdated($batch));
 })
 ->then(function (Batch $batch) {
    // All jobs succeeded
    Notification::send($user, new CsvImportComplete($batch->id));
 })
 ->catch(function (Batch $batch, Throwable $e) {
    // First job failure
    Log::error('CSV batch failed', ['batch_id' => $batch->id, 'error' => $e->getMessage()]);
 })
 ->finally(function (Batch $batch) {
    // Always called — success or failure
    Storage::delete($file->path);
 })
 ->onQueue('imports')
 ->dispatch();

return ['batch_id' => $batch->id];   // give the user a way to poll progress
```

By default, **a single failure cancels the rest of the batch.** Use `allowFailures()` to keep going (see next rule).

## Mixing them — chain that contains batches

```php
// ✅ Step 1 (single), then step 2 (parallel batch), then step 3 (single)
Bus::chain([
    new FlushPodcastCache(42),
    Bus::batch([
        new ReleasePodcast(42, region: 'us-east'),
        new ReleasePodcast(42, region: 'eu-west'),
        new ReleasePodcast(42, region: 'ap-south'),
    ]),
    new NotifyPodcastReleased(42),
])->dispatch();
```

This runs:
1. `FlushPodcastCache` first
2. Then three `ReleasePodcast` jobs in parallel (batch waits for all to finish)
3. Then `NotifyPodcastReleased` after the batch completes

## Incorrect

```php
// ❌ Using chain when jobs are independent — slow

Bus::chain([
    new SendOrderConfirmation(1),   // independent
    new SendOrderConfirmation(2),   // independent
    new SendOrderConfirmation(3),   // independent
    // ... 100 more
])->dispatch();
// Result: 100 emails sent one at a time. The 100th waits ~5 minutes.
// Should have been Bus::batch — parallel.
```

```php
// ❌ Using batch when there's a dependency — race condition

Bus::batch([
    new CreateInvoice($orderId),   // creates invoice row
    new EmailInvoiceLink($orderId), // needs the invoice row to exist
])->dispatch();
// Race: EmailInvoiceLink may run before CreateInvoice finishes. Email links to a row
// that doesn't exist yet. Should have been Bus::chain.
```

## Detection

```bash
# Find Bus::chain with many jobs — likely should be Bus::batch
grep -rEnA20 'Bus::chain\(' --include='*.php' app/ \
  | awk '/Bus::chain/,/\)\->/{print}' | grep -c 'new '

# Find Bus::batch where dependency between jobs is implied by name
# (manual review — look at the job names in batch arrays)
grep -rEnA5 'Bus::batch\(' --include='*.php' app/ | head -30
```

Reference: [Laravel 13 — Queues: Job Chaining](https://laravel.com/docs/13.x/queues#job-chaining) · [Laravel 13 — Queues: Job Batching](https://laravel.com/docs/13.x/queues#job-batching)

---


## Batch Failure Handling — allowFailures() and Callbacks

**Impact: HIGH (Default batch behaviour cancels remaining jobs on first failure; allowFailures() keeps them running)**

`Bus::batch` has an opinion about failures: **by default, a single job failure cancels the rest of the batch.** Any unstarted jobs in the batch are silently dropped; in-flight jobs complete but don't trigger further dispatches.

For a "process these 1000 CSV rows" batch where one row failing shouldn't stop the other 999, that default is wrong. Use `allowFailures()`.

For a "create these 3 resources" batch where if any fails, the others are pointless, the default IS right.

## Default: fail-fast

```php
// ✅ Fail-fast: if one job fails, cancel the rest

Bus::batch([
    new CreateOrderRecord($data),
    new ReserveInventory($data),
    new ChargePayment($data),
])->catch(function (Batch $batch, Throwable $e) {
    // Triggered on the FIRST failure; remaining jobs cancelled
    OrderFlow::rollback($data);
})->dispatch();
```

Right call here: you don't want `ChargePayment` to run if `ReserveInventory` failed.

## allowFailures(): keep going

```php
// ✅ Resilient batch: one row failing doesn't stop the import

$batch = Bus::batch(
    $rows->map(fn ($row) => new ImportCsvRow($row))
)->name('CSV import')
 ->allowFailures()                            // ← critical
 ->then(function (Batch $batch) {
    // All processed (some may have failed)
    $report = "Imported {$batch->processedJobs()} / {$batch->totalJobs}; failed: {$batch->failedJobs}";
    Mail::to($user)->send(new CsvImportReport($report));
 })
 ->dispatch();
```

`allowFailures()` semantics:
- Failed jobs go to `failed_jobs` (same as a normal job failure)
- The batch's `then` callback fires when all jobs have either succeeded OR failed
- The batch's `catch` callback NEVER fires (since failures are allowed)
- `$batch->failedJobs` and `$batch->processedJobs()` show the split

## Querying batch state

For UI / monitoring:

```php
$batch = Bus::findBatch($batchId);

$batch->pendingJobs;         // not yet processed
$batch->totalJobs;           // total count
$batch->processedJobs();     // pendingJobs done (success + fail)
$batch->failedJobs;          // count of failures
$batch->progress();          // 0-100 percentage
$batch->finished();          // bool: all jobs processed
$batch->cancelled();         // bool: cancel() was called
```

Use this to render a progress page:

```php
// routes/web.php
Route::get('/batches/{id}/progress', function (string $id) {
    $batch = Bus::findBatch($id);
    abort_unless($batch, 404);
    return [
        'total' => $batch->totalJobs,
        'done' => $batch->processedJobs(),
        'failed' => $batch->failedJobs,
        'progress' => $batch->progress(),
        'finished' => $batch->finished(),
    ];
});
```

## Cancelling a batch

```php
// Cancel any remaining queued jobs (in-flight jobs still complete)
Bus::findBatch($batchId)->cancel();

// In a job, check before starting work
public function handle(): void
{
    if ($this->batch()->cancelled()) {
        return;
    }
    // do the work
}
```

Useful pattern: a user clicks "cancel import" on the progress page, you call `cancel()`, jobs see the cancelled state and exit early.

## Incorrect — common mistakes

```php
// ❌ Using catch but not realising it doesn't fire if allowFailures()
Bus::batch($jobs)
    ->allowFailures()
    ->catch(function (Batch $batch, Throwable $e) {
        // NEVER FIRES because allowFailures() is set
        // (failed jobs go to failed_jobs, but the batch carries on)
    })
    ->dispatch();
```

```php
// ❌ Forgetting allowFailures on a "best effort" batch
Bus::batch($csvJobs)->dispatch();   // first failure → remaining cancelled
// Customer reported one row failed, 5000 others never imported.
```

```php
// ❌ Trying to "retry the batch" by re-dispatching all jobs
// Bus::batch doesn't have built-in retry. If you want retry semantics,
// either:
//   - Set $tries on the individual jobs (each retries independently)
//   - Filter for failed jobs after the batch and dispatch a NEW batch with just those
```

## When the batch's `then` should run

`then` callback fires when:
- All jobs succeeded (no failures); OR
- `allowFailures()` is set AND all jobs have completed (some passed, some failed)

`catch` fires when:
- A job fails AND `allowFailures()` is NOT set (then cancels remaining)

`finally` always fires after `then` or `catch`.

## Detection

```bash
# Batches without explicit failure mode choice — manual review
grep -rEnA10 'Bus::batch\(' --include='*.php' app/ | grep -B2 -A5 '->dispatch()'

# Look for batches that DON'T have a callback (then/catch/finally) — fire-and-forget batches
# are usually fine, but worth checking they're intentional
```

Reference: [Laravel 13 — Queues: Allowing Failures](https://laravel.com/docs/13.x/queues#batch-failures) · [Laravel 13 — Queues: Inspecting Batches](https://laravel.com/docs/13.x/queues#inspecting-batches)

---


## Chunking Large Datasets — Don't Dispatch One Job Per Row

**Impact: HIGH (Dispatching 100k jobs at once floods Redis, overwhelms workers, and starves SLA traffic)**

For "process 100k orders" or "send emails to all users", the naive approach is one job per record. With 100k jobs:

- Dispatch itself blocks for seconds (network round-trips to Redis)
- The queue fills with 100k entries; other jobs wait their turn
- Each job has overhead (~10-50ms even for trivial work) — totalling 16-80 minutes regardless of worker count
- Failed-jobs table can grow by tens of thousands of rows if a downstream service hiccups

**Chunk it.** Process N items per job. Dispatch fewer, larger jobs. Get a 10×-100× efficiency gain.

## Incorrect

```php
// ❌ One job per record at scale
$users = User::all();   // 100,000 rows loaded into memory
foreach ($users as $user) {
    SendMarketingEmail::dispatch($user->id);
}
// 100,000 dispatches; takes ~30s just to enqueue; queue worker overhead dominates
```

## Correct — chunk inside a Bus::batch

```php
// ✅ Chunk to 500 rows per job; ~200 batch jobs instead of 100k

use App\Jobs\SendMarketingEmailChunk;
use Illuminate\Support\Facades\Bus;

$batch = Bus::batch([])
    ->name('Marketing send — Q3 newsletter')
    ->allowFailures()
    ->then(function ($batch) use ($campaign) {
        $campaign->markComplete();
    })
    ->dispatch();

User::where('subscribed', true)
    ->select('id')
    ->chunkById(500, function ($chunkOfUsers) use ($batch) {
        $batch->add(new SendMarketingEmailChunk(
            userIds: $chunkOfUsers->pluck('id')->all()
        ));
    });
```

```php
// The chunk job processes its slice
class SendMarketingEmailChunk implements ShouldQueue
{
    use Dispatchable, Queueable;

    /** @param int[] $userIds */
    public function __construct(public readonly array $userIds) {}

    public function handle(): void
    {
        User::whereIn('id', $this->userIds)
            ->select(['id', 'email', 'name'])
            ->chunkById(100, function ($users) {
                foreach ($users as $user) {
                    Mail::to($user)->send(new MarketingEmail($user));
                }
            });
    }
}
```

**Wins:**
- 200 batch jobs instead of 100,000 dispatches
- Each batch job processes 500 users in <30 seconds
- Total throughput: same; queue overhead: drastically reduced
- Batch progress tracking shows "180 / 200 chunks done" instead of "84,000 / 100,000"

## Chunk size — picking N

| Job characteristic | Chunk size |
|---|---|
| Lightweight (in-memory transformation) | 1000–5000 |
| External API per item (Stripe, etc.) | 50–200 |
| Email sends | 100–500 |
| DB writes per item | 200–1000 |
| Complex business logic per item | 25–100 |

Aim for **each chunk job running 10s–60s.** Shorter and queue overhead dominates; longer and you lose recoverability (a single failed item burns the whole chunk).

## chunkById vs cursor()

```php
// ✅ chunkById — most efficient for "iterate all rows"
User::query()->select('id')->chunkById(500, fn ($chunk) => /* ... */);

// ✅ cursor() — for streaming inside a chunk (one record at a time, low memory)
foreach (User::cursor() as $user) { /* ... */ }   // 100k records, ~10MB memory

// ❌ All() — loads everything into memory
foreach (User::all() as $user) { /* ... */ }      // 100k records, ~1.5GB memory
```

Pattern: use `chunkById` at the **dispatch** site to build the batch; use `chunkById` again inside the chunk job for memory efficiency processing those 500.

## Idempotency in chunk jobs

A chunk job may run twice (the queue's at-least-once guarantee). Make the chunk's effect idempotent:

```php
public function handle(): void
{
    User::whereIn('id', $this->userIds)
        ->whereDoesntHave('emailLogs', function ($q) {
            $q->where('campaign_id', $this->campaignId);
        })
        ->each(function ($user) {
            Mail::to($user)->send(new MarketingEmail($user));
            $user->emailLogs()->create([
                'campaign_id' => $this->campaignId,
                'sent_at' => now(),
            ]);
        });
}
```

The `whereDoesntHave` filter ensures the email is sent at most once per user per campaign, even if the chunk job retries.

## Don't ship Eloquent collections in the constructor

```php
// ❌ Passes a Collection — entire object serialised
new SendMarketingEmailChunk($users);

// ✅ Pass IDs — serialised as int[]
new SendMarketingEmailChunk(userIds: $users->pluck('id')->all());
```

See [`design-pass-ids-not-models`](design-pass-ids-not-models.md).

## Detection

```bash
# Suspicious patterns — foreach + dispatch
grep -rEnB1 -A2 'foreach\s*\(.*\$\w+\s+as\s+\$\w+\)' --include='*.php' app/ | \
  grep -B1 'dispatch\('
# Manual review: anything iterating > 100 records and dispatching one job each
```

Reference: [Laravel 13 — Queues: Adding Jobs to Batches](https://laravel.com/docs/13.x/queues#adding-jobs-to-batches) · [Laravel 13 — Eloquent: Chunking Results](https://laravel.com/docs/13.x/eloquent#chunking-results)

---


## Queue::fake / Bus::fake — Testing Dispatch Behaviour

**Impact: HIGH (Without queue faking, tests either hit real queues (slow, brittle) or skip queue assertions entirely)**

When testing code that dispatches jobs, you want to verify "this controller dispatches `ChargeOrder` with order ID 42" — not actually run the queue. Laravel provides two faking facades:

- **`Queue::fake()`** — intercepts `dispatch()` calls; assert on what was pushed
- **`Bus::fake()`** — intercepts `Bus::dispatch()`, `Bus::chain()`, `Bus::batch()`; assert on what was dispatched/chained/batched

For testing dispatched jobs from controllers/services, use `Queue::fake()`. For testing chains and batches, use `Bus::fake()`.

## Queue::fake — assert single job dispatches

```php
use App\Jobs\ChargeOrder;
use Illuminate\Support\Facades\Queue;

test('checkout dispatches the charge job', function () {
    Queue::fake();

    $order = Order::factory()->create();

    $this->postJson('/checkout', ['order_id' => $order->id])
        ->assertOk();

    // ✅ Assert ChargeOrder was dispatched at least once
    Queue::assertPushed(ChargeOrder::class);

    // ✅ Assert it was dispatched with specific args
    Queue::assertPushed(ChargeOrder::class, function (ChargeOrder $job) use ($order) {
        return $job->orderId === $order->id;
    });

    // ✅ Assert it was dispatched to the 'high' queue
    Queue::assertPushedOn('high', ChargeOrder::class);

    // ✅ Assert it was NOT pushed (negative case)
    Queue::assertNotPushed(RefundOrder::class);

    // ✅ Assert exactly N dispatches
    Queue::assertPushed(ChargeOrder::class, 1);   // exactly once
});
```

**`Queue::fake()` semantics:**
- After fake, `ChargeOrder::dispatch(...)` records a "push" event, does NOT actually queue or run
- Assertions read those records
- Job `handle()` is never called
- The controller / service code under test runs normally; only the dispatch is faked

## Bus::fake — for chains and batches

```php
use App\Jobs\ProcessPodcast;
use App\Jobs\OptimizePodcast;
use Illuminate\Support\Facades\Bus;

test('podcast workflow dispatches the right chain', function () {
    Bus::fake();

    $this->postJson('/podcasts', ['title' => 'Test'])
        ->assertOk();

    // ✅ Assert a chain was dispatched containing these jobs in order
    Bus::assertChained([
        ProcessPodcast::class,
        OptimizePodcast::class,
        ReleasePodcast::class,
    ]);

    // ✅ Assert specific batches
    Bus::assertBatched(function ($batch) {
        return $batch->jobs->count() === 3
            && $batch->name === 'CSV import';
    });

    // ✅ Single dispatches
    Bus::assertDispatched(SendNotification::class);
});
```

## When to use which

| Code under test dispatches… | Use |
|---|---|
| Plain `Job::dispatch(...)` | `Queue::fake()` |
| `dispatch(new Job(...))` (functional helper) | `Queue::fake()` OR `Bus::fake()` (both work) |
| `Bus::dispatch(...)` (facade) | `Bus::fake()` |
| `Bus::chain([...])` | `Bus::fake()` |
| `Bus::batch([...])` | `Bus::fake()` |

**Rule of thumb:** if the controller uses `Bus::*` anywhere, use `Bus::fake()`. Otherwise `Queue::fake()` is simpler.

## Fake only certain jobs

```php
// ✅ Fake some jobs, let others run normally
Queue::fake([
    ChargeOrder::class,            // intercept this one
    SendOrderConfirmation::class,  // and this one
    // Anything else dispatches normally (caution: could touch real queue in tests)
]);
```

Useful when:
- Most jobs should "actually run" via `Queue::push` in a sync driver for integration tests
- A specific job (e.g., one that hits Stripe) needs interception

## Testing the job's `handle()` directly

Sometimes you don't care about dispatch; you care about the job's logic. Test it directly:

```php
test('ChargeOrder marks the order paid', function () {
    Http::fake(['api.stripe.com/*' => Http::response(['id' => 'ch_123'])]);

    $order = Order::factory()->create(['status' => 'pending']);

    (new ChargeOrder($order->id))->handle(new StripeGateway());

    expect($order->fresh()->status)->toBe('paid');
    expect($order->fresh()->stripe_charge_id)->toBe('ch_123');
});
```

No `Queue::fake()` needed — you're calling `handle()` directly. This is the right level for testing the job's business logic.

## Testing `failed()`

```php
test('ChargeOrder marks order failed on permanent error', function () {
    Http::fake(['api.stripe.com/*' => Http::response(['error' => 'card_declined'], 402)]);

    $order = Order::factory()->create(['status' => 'pending']);
    $job = new ChargeOrder($order->id);

    // Run the failed handler with an exception
    $job->failed(new CardException('card_declined'));

    expect($order->fresh()->status)->toBe('payment_failed');
    Notification::assertSentTo($order->user, OrderPaymentFailed::class);
});
```

## Common bugs

```php
// ❌ Forgetting Queue::fake() — test actually hits the queue
test('checkout dispatches charge', function () {
    $this->postJson('/checkout', ['order_id' => 1]);
    // → ChargeOrder actually queued; the next worker picks it up; test may pass even if charge is wrong
});

// ❌ Asserting after running the test logic, but Queue::fake was set BEFORE
Queue::assertPushed(...);   // empty before any code runs

// ❌ Mixing Queue::fake and Bus::fake (only one wins; later overrides earlier)
Queue::fake();
Bus::fake();
// → Queue::fake is no longer effective; assertions on Queue::* will fail
```

## Detection

```bash
# Tests that dispatch jobs but don't fake the queue
grep -rEln 'dispatch\(' --include='*Test.php' --include='*.php' tests/ | while read f; do
  grep -qE 'Queue::fake|Bus::fake' "$f" || echo "NO FAKE: $f"
done

# Tests that fake but never assert (smoke tests, possibly OK but worth checking)
grep -rEln 'Queue::fake|Bus::fake' tests/ | while read f; do
  grep -qE 'Queue::assert|Bus::assert' "$f" || echo "FAKE WITHOUT ASSERT: $f"
done
```

Reference: [Laravel 13 — Queues: Testing](https://laravel.com/docs/13.x/queues#testing) · [Laravel 13 — Bus Mocking](https://laravel.com/docs/13.x/mocking#bus-fake)

---


## Scheduled Jobs — Schedule::job + withoutOverlapping

**Impact: HIGH (Scheduled tasks that run too long can pile up; long-running scheduled work should queue, not block scheduler)**

The Laravel scheduler is a single cron process that fires every minute and dispatches scheduled tasks. Two failure modes commonly bite:

1. **Long-running tasks run inline** — a 3-minute import scheduled `everyMinute()` overlaps itself; `withoutOverlapping()` is missing
2. **The scheduler itself blocks** — `Schedule::call(...)` runs the closure in the scheduler process, blocking the next tick

Both fix the same way: **dispatch long work to the queue, use `withoutOverlapping()` for safety.**

## Three scheduling APIs — choose carefully

```php
// app/Console/Kernel.php (Laravel 11+: routes/console.php)
use Illuminate\Support\Facades\Schedule;

// ❌ 1. Inline closure — runs in the scheduler process
Schedule::call(function () {
    Customer::all()->each(fn ($c) => recalculateLoyalty($c));   // 5 min of work
})->hourly();
// If this overruns the hour, the next tick is delayed. Bad.

// ✅ 2. Dispatch a queued job
Schedule::job(new RecalculateAllLoyalty)->hourly();
// → Job pushed to queue; worker picks it up; scheduler tick takes <1ms.

// ✅ 3. Artisan command (queued if the command queues a job)
Schedule::command('app:recalculate-loyalty')->hourly();
// Pattern: the command dispatches a job; tests both the command and the job.
```

**Use `Schedule::job(...)`** for any task that takes more than a few seconds. The scheduler hands off; the queue worker does the work.

## withoutOverlapping — prevent piled-up runs

```php
// ✅ Self-overlap prevention
Schedule::job(new GenerateDailyReport)
    ->dailyAt('02:00')
    ->withoutOverlapping(30);   // lock expires after 30 minutes (in case worker dies)
```

`withoutOverlapping()`:
- Acquires a mutex (via the configured cache) before running
- If another instance is already running, the new tick is skipped
- The mutex expires after the given minutes (default 24 hours — usually too long; set explicitly)

**Always set an explicit expiry.** If a worker dies mid-task without releasing the mutex, the next 24-hour expiry default means the task is skipped for a full day.

## Pattern: full setup for a scheduled batch

```php
// routes/console.php

use App\Jobs\NightlyOrderReconciliation;
use Illuminate\Support\Facades\Schedule;

Schedule::job(new NightlyOrderReconciliation, 'low')   // 2nd arg = queue (positional, not chained)
    ->dailyAt('03:00')
    ->withoutOverlapping(120)                          // 2-hour mutex; previous run done by then
    ->onFailure(fn () => Slack::alert('Nightly reconciliation failed to schedule'))
    ->onSuccess(fn () => Log::info('Nightly reconciliation scheduled'));
```

Each modifier:
- **`Schedule::job($job, $queue, $connection)`** — queue and connection are **positional args**, not chained methods. `Schedule::job(...)->onQueue(...)` does not exist and throws `BadMethodCallException`.
- `withoutOverlapping(120)` — prevent double-runs if one tick runs long
- `onFailure` / `onSuccess` — hooks for monitoring the *dispatch* (not the job's run)
- **Don't chain `runInBackground()` on `Schedule::job(...)`** — it's documented to apply only to `Schedule::command(...)` and `Schedule::exec(...)`. The job already queues, so backgrounding the dispatch tick is meaningless.

## Common bugs

```php
// ❌ Long closure in the scheduler — blocks subsequent ticks
Schedule::call(function () {
    Mail::to(User::all())->send(new WeeklyDigest);   // 6-min send loop
})->weekly();

// ❌ No overlap prevention on a frequently-running long job
Schedule::call(fn () => recalculateMetrics())->everyMinute();
// If recalculateMetrics() ever takes > 60s, it overlaps with itself.

// ❌ Heavy job dispatched without onQueue routing — competes with user traffic
Schedule::job(new MassivelyHeavyJob)->dailyAt('02:00');
// Goes to 'default'. Lands amid morning traffic if east-coast users wake up.

// ❌ Schedule::call() for what should be a job
Schedule::call(function () {
    foreach (Order::pending() as $order) {
        ChargeOrder::dispatch($order->id);
    }
})->everyMinute();
// Better:
Schedule::call(fn () =>
    Order::pending()->each(fn ($o) => ChargeOrder::dispatch($o->id))
)->everyMinute()->withoutOverlapping(5);
// Even better: the scheduler runs cheaply; the jobs queue and run in parallel.
```

## Running the scheduler

The scheduler needs a cron entry on the server:

```bash
# crontab -e
* * * * * cd /var/www/app && php artisan schedule:run >> /dev/null 2>&1
```

Or on Forge / Vapor: handled by the platform. Verify it's actually configured — a silent missing cron means no scheduled jobs ever run.

## Detection

```bash
# Find Schedule::call with long closures (closure-style scheduled tasks doing real work)
grep -rEnA10 'Schedule::call\(' --include='*.php' routes/console.php app/Console/

# Schedule::job without withoutOverlapping
# Note: `grep -L` doesn't work meaningfully on a pipe (operates on stdin as one stream).
# Use a per-file loop instead.
for f in routes/console.php $(find app/Console -name '*.php' 2>/dev/null); do
  [ -f "$f" ] || continue
  grep -qE 'Schedule::job\(' "$f" && ! grep -qE 'withoutOverlapping' "$f" \
    && echo "MISSING withoutOverlapping: $f"
done

# Verify cron is set on the server (manual)
crontab -l | grep schedule:run
```

Reference: [Laravel 13 — Task Scheduling: Defining Schedules](https://laravel.com/docs/13.x/scheduling#defining-schedules) · [Laravel 13 — Preventing Task Overlaps](https://laravel.com/docs/13.x/scheduling#preventing-task-overlaps)

---


## When to Adopt Horizon

**Impact: MEDIUM (Horizon helps on Redis at scale; on database queue or small apps, it adds complexity without payoff)**

[Laravel Horizon](https://laravel.com/docs/13.x/horizon) is a Redis-only queue dashboard + worker manager. It provides:

- A web dashboard showing queue throughput, wait times, recent jobs, failed jobs
- Auto-scaling workers (allocate processes between queues based on load)
- Metrics on job runtime, throughput, failure rate
- Monitoring of specific job types

Horizon **only works with Redis.** If you're on the `database` queue driver, Horizon isn't an option. If you're on `redis` but small (single worker, low throughput), Horizon is more setup than it earns back.

## When Horizon is worth adopting

Adopt Horizon when **all** of these are true:

- Queue driver is `redis` (Horizon requirement)
- Multiple worker servers OR many workers on one server (3+ workers makes the auto-balancing meaningful)
- You want a visible dashboard (current queue state, recent failures, throughput trends)
- Job-runtime metrics matter to your team
- You're comfortable maintaining a Horizon process alongside the workers

## When Horizon is overkill

Skip Horizon when:

- You're on the `database` queue driver (not supported)
- A single worker is enough (auto-balancing doesn't help)
- You don't need a dashboard (you trust Supervisor + `failed_jobs` + logs)
- The team doesn't want another moving part
- You're on Laravel Vapor (uses SQS, not Redis; Vapor has its own dashboard)

For most apps starting out: **don't adopt Horizon on day one.** Plain `queue:work` + Supervisor is enough. Adopt later when the dashboard or auto-balancing is concretely needed.

## Setup (if adopting)

```bash
composer require laravel/horizon
php artisan horizon:install
```

Configure `config/horizon.php`:

```php
'environments' => [
    'production' => [
        'supervisor-high' => [
            'queue' => ['high'],
            'balance' => 'simple',
            'processes' => 4,
            'timeout' => 60,
            'tries' => 3,
        ],
        'supervisor-default' => [
            'queue' => ['high', 'default', 'low'],
            'balance' => 'auto',
            'minProcesses' => 2,
            'maxProcesses' => 12,
            'timeout' => 60,
            'tries' => 3,
        ],
    ],
    'local' => [
        'supervisor-1' => [
            'queue' => ['high', 'default', 'low'],
            'balance' => 'simple',
            'processes' => 2,
            'timeout' => 60,
            'tries' => 1,
        ],
    ],
],
```

Run Horizon in production (replace `queue:work` Supervisor entries with a single Horizon entry):

```ini
[program:laravel-horizon]
process_name=%(program_name)s
command=php /var/www/app/artisan horizon
autostart=true
autorestart=true
user=forge
redirect_stderr=true
stdout_logfile=/var/www/app/storage/logs/horizon.log
stopwaitsecs=3600
```

Horizon spawns its own worker subprocesses based on the config. **Replace** plain `queue:work` Supervisor entries — don't run both.

## Balancing strategies

| Strategy | Behaviour | Use when |
|---|---|---|
| `simple` | Equal split across queues | Predictable load |
| `auto` | Dynamically reassign processes based on each queue's wait time | Variable load between queues |
| `false` | All workers process all queues from `queue` array | Single queue, no priority |

For most production apps with high/default/low: `auto` on the general supervisor, `simple` on the `high`-only supervisor.

## Dashboard security

The Horizon dashboard is publicly accessible by default. Lock it down:

```php
// app/Providers/HorizonServiceProvider.php
protected function gate(): void
{
    Gate::define('viewHorizon', function ($user) {
        return $user->is_admin || in_array($user->email, [
            'admin@example.com',
        ]);
    });
}
```

In `config/app.php`, register the gate in production.

## Tags — for filtering in the dashboard

Tag jobs to filter them in the Horizon UI:

```php
class ChargeOrder implements ShouldQueue
{
    use Dispatchable, Queueable;

    public function __construct(public readonly int $orderId) {}

    public function tags(): array
    {
        return ['order:' . $this->orderId, 'payment'];
    }

    public function handle(): void { /* ... */ }
}
```

Now in the Horizon dashboard, you can search for `order:42` and see every job related to that order.

## Detection

```bash
# Is Horizon installed?
grep -E 'laravel/horizon' composer.json

# If installed, is it actually running as the worker (not plain queue:work)?
grep -nH 'horizon\b' /etc/supervisor/conf.d/*.conf supervisor*.conf 2>/dev/null

# Horizon installed but never started → dashboards empty, workers don't run
# Plain queue:work running alongside Horizon → both compete; jobs run twice
```

Reference: [Laravel 13 — Horizon Introduction](https://laravel.com/docs/13.x/horizon) · [Laravel 13 — Horizon: Balancing Strategies](https://laravel.com/docs/13.x/horizon#balancing-strategies)

---

