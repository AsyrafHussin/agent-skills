---
title: failed(Throwable $e) — Handle Permanent Failures
impact: HIGH
impactDescription: "Without failed(), permanent failures are silent; customers see orders stuck in 'pending'"
tags: retry, failed, failure-handling
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
