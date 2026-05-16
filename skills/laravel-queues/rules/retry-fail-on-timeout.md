---
title: "#[FailOnTimeout] — Don't Burn Attempts on Hangs"
impact: HIGH
impactDescription: "A hung HTTP call kills 5 attempts at 30s each before failing — 2.5 min of wasted worker time per stuck job"
tags: retry, timeout, attributes
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
