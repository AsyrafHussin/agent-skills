---
title: "after_commit: true — Wait for DB Transaction Before Dispatching"
impact: CRITICAL
impactDescription: "Without this, queued jobs reference rows that don't exist yet; jobs fail with ModelNotFoundException at random"
tags: config, after-commit, transactions, race-condition
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
