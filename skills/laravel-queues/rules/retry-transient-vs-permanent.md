---
title: Transient vs Permanent Failures — release() or throw?
impact: HIGH
impactDescription: "Treating every error as retryable wastes attempts; treating none as retryable fails on transient blips"
tags: retry, transient, permanent, release
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
