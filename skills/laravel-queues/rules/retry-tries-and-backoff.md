---
title: $tries and #[Backoff] — Configure Both
impact: HIGH
impactDescription: "Default 3 tries with no backoff causes retry storms; misconfigured backoff means slow recovery"
tags: retry, tries, backoff, attributes
---

## $tries and #[Backoff] — Configure Both

**Impact: HIGH (Default 3 tries with no backoff causes retry storms; misconfigured backoff means slow recovery)**

Laravel's default retry behaviour is conservative: 1 attempt unless you set `$tries` (or the `--tries` flag on `queue:work`). That's almost never enough for jobs that touch external services — Stripe blips, network glitches, third-party rate limits all warrant a retry. But retrying immediately with no delay is just as bad — you hammer the same failing dependency 5 times in a row.

**Set both:** `$tries` (or `--tries` flag) and `#[Backoff(...)]` (or `$backoff` property) for delays. Use **exponential delays** for transient failures.

## Incorrect

```php
// ❌ Default 1 attempt — any transient failure is fatal
class ChargeCustomer implements ShouldQueue { /* … */ }
// No $tries → 1 attempt → first Stripe blip = job marked failed

// ❌ Many tries with no backoff — retry storm
class ChargeCustomer implements ShouldQueue
{
    public int $tries = 5;
    // No backoff → 5 immediate retries → if Stripe is rate-limiting you, you're rate-limited worse
}

// ❌ Constant backoff = naive
class ChargeCustomer implements ShouldQueue
{
    public int $tries = 5;
    public int $backoff = 30;
    // 5 retries × 30s apart = 150s recovery on a service that's been down for 5 minutes
}
```

## Correct — Laravel 13 idiom: `#[Backoff]` attribute with exponential delays

```php
// ✅ Production-grade retry config

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\Backoff;

#[Backoff([1, 5, 30, 120, 600])]   // exponential-ish: 1s, 5s, 30s, 2min, 10min
class ChargeCustomer implements ShouldQueue
{
    use Dispatchable, Queueable;

    public int $tries = 5;
    public int $timeout = 30;
    public int $maxExceptions = 3;   // stop retrying after 3 different exceptions

    // …
}
```

**What each setting does:**

| Setting | Effect |
|---|---|
| `$tries = 5` | Job will be released back to the queue up to 4 times before being marked failed |
| `#[Backoff([1, 5, 30, 120, 600])]` | Delay between retries: 1s, 5s, 30s, 2min, 10min |
| `$timeout = 30` | Each attempt has 30s before being killed (release or fail depending on `FailOnTimeout`) |
| `$maxExceptions = 3` | Even if `$tries` is 5, stop after 3 *distinct* exception types — protects against catastrophic bugs |

## Property vs attribute

Both forms work in Laravel 13. The attribute (`#[Backoff(...)]`) is preferred for new code — cleaner, sits next to the class declaration:

```php
// ✅ Attribute (preferred in Laravel 13)
#[Backoff([1, 5, 30])]
class X implements ShouldQueue {}

// ✅ Property (still works)
class X implements ShouldQueue {
    public int|array $backoff = [1, 5, 30];
}

// ✅ Computed backoff via method
public function backoff(): array
{
    return [1, 5, 30 * $this->attempts()];   // dynamic
}
```

## What value should `$tries` be?

| Job type | Recommended `$tries` |
|---|---|
| Payment / external charge | 5–7 (with idempotency) |
| External API call (Stripe, Shopify, etc.) | 3–5 |
| Email / notification | 3 |
| Internal pure computation | 1 (don't retry on real bugs) |
| Data export / batch processing | 3 (with chunking) |

For internal-only jobs (no external dependencies), `$tries = 1` is often right — retrying on a programmer error is just wasted compute.

## Setting at the worker level vs the job level

```bash
# Worker-level defaults (apply to all jobs unless overridden)
php artisan queue:work --tries=3 --backoff=3 --max-time=3600
```

Job-level settings (`$tries`, `#[Backoff]`) override worker-level. Use **job-level** for production — different jobs need different policies. Use **worker-level** as a sensible default for unconfigured jobs.

## Detection

```bash
# Jobs without $tries or #[Tries] / #[Backoff]
for f in app/Jobs/*.php; do
  if grep -q 'implements ShouldQueue' "$f" && ! grep -qE '\$tries|#\[Tries|#\[Backoff' "$f"; then
    echo "NO RETRY CONFIG: $f"
  fi
done

# Find jobs with constant backoff (= 0 or a single int) on external-facing work
grep -rEn 'public int \$backoff = [0-9]+;' --include='*.php' app/Jobs/
```

Reference: [Laravel 13 — Queues: Max Job Attempts & Timeout](https://laravel.com/docs/13.x/queues#max-job-attempts-and-timeout) · [Laravel 13 — Queues: Backoff Configuration](https://laravel.com/docs/13.x/queues#max-job-attempts-and-timeout)
