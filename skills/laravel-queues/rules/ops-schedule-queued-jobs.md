---
title: Scheduled Jobs — Schedule::job + withoutOverlapping
impact: HIGH
impactDescription: "Scheduled tasks that run too long can pile up; long-running scheduled work should queue, not block scheduler"
tags: ops, scheduling, withoutOverlapping
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
