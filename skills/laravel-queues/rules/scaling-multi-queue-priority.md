---
title: Multi-Queue Priority Lanes — high/default/low
impact: HIGH
impactDescription: "Without priority lanes, a flood of low-priority emails stalls SLA-critical payment jobs"
tags: scaling, queues, priority, sla
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
