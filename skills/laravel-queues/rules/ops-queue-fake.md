---
title: Queue::fake / Bus::fake — Testing Dispatch Behaviour
impact: HIGH
impactDescription: "Without queue faking, tests either hit real queues (slow, brittle) or skip queue assertions entirely"
tags: testing, queue-fake, bus-fake, assertions
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
