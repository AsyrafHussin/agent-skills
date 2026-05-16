---
title: Idempotency — Safe to Re-run Without Duplicates
impact: CRITICAL
impactDescription: "Without idempotency, retries duplicate payments, send double emails, and create phantom rows"
tags: design, idempotency, retries, payments
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
