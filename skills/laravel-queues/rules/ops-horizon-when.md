---
title: When to Adopt Horizon
impact: MEDIUM
impactDescription: "Horizon helps on Redis at scale; on database queue or small apps, it adds complexity without payoff"
tags: ops, horizon, redis
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
