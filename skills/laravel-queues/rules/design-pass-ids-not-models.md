---
title: Pass IDs to Jobs, Not Eloquent Models
impact: CRITICAL
impactDescription: "Serialised models go stale; large payloads inflate the queue; SerializesModels refresh-from-DB has subtle traps"
tags: design, serialization, models
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
