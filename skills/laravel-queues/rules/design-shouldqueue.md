---
title: implements ShouldQueue — The #1 Production Bug
impact: CRITICAL
impactDescription: "Without ShouldQueue, the job runs synchronously inside the request — what was meant as 'queued' blocks HTTP"
tags: design, shouldqueue, sync, ai-fingerprint
---

## implements ShouldQueue — The #1 Production Bug

**Impact: CRITICAL (Without ShouldQueue, the job runs synchronously inside the request — what was meant as 'queued' blocks HTTP)**

The single most common Laravel queue bug: a class that uses `Dispatchable` and looks like a job, but doesn't implement `ShouldQueue`. Without that one interface marker, `dispatch()` calls `handle()` **inline**, blocking whatever called it. The 800ms "queued" Stripe charge isn't queued — it's adding 800ms to the checkout request.

AI-generated job scaffolding sometimes drops the `implements ShouldQueue` or names a "service" class as if it were a job. Always verify.

## Incorrect

```php
// ❌ No ShouldQueue — looks queued, runs sync

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class SendWelcomeEmail
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(public readonly int $userId) {}

    public function handle(): void
    {
        $user = User::findOrFail($this->userId);
        Mail::to($user)->send(new WelcomeEmail($user));   // takes 600ms
    }
}

// Controller:
public function register(Request $request)
{
    $user = User::create($request->validated());

    SendWelcomeEmail::dispatch($user->id);   // ← runs SYNCHRONOUSLY (no ShouldQueue!)

    return response()->json(['user' => $user]);
}
```

**What happens:**
- `dispatch()` checks: does this class implement `ShouldQueue`? **No.**
- Laravel: "OK, calling `handle()` immediately."
- 600ms email send happens inside the HTTP request
- User sees a 600ms+ register endpoint

**Why this happens:**
- Job class scaffolded but `implements ShouldQueue` forgotten
- Copy-pasted from a "Dispatchable" helper trait example
- AI generated a class that looks like a job but isn't

## Correct

```php
// ✅ Implements ShouldQueue — actually queues

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class SendWelcomeEmail implements ShouldQueue   // ← critical
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(public readonly int $userId) {}

    public function handle(): void { /* ... */ }
}

// Controller — same code, now actually async
SendWelcomeEmail::dispatch($user->id);
```

## Sanity check during a code review

When you see `Dispatchable` + `handle()`, immediately scan for `implements ShouldQueue`. If it's missing, that's the bug.

## The "dispatchSync intentionally" exception

Sometimes you genuinely want a job-style class to run sync — e.g., a complex orchestration you want to call from both controllers and queued contexts. Two acceptable patterns:

```php
// ✅ Always sync — don't pretend to be a job
final class CalculateInvoiceTotals  // no Dispatchable, no ShouldQueue
{
    public function __invoke(Invoice $invoice): Money { /* ... */ }
}
```

```php
// ✅ Sync OR queued, caller's choice — but the class IS a queued job
final class CalculateInvoiceTotals implements ShouldQueue { /* ... */ }

// To run sync from a controller:
CalculateInvoiceTotals::dispatchSync($invoice->id);
```

The bug is the *third* option — a class that *looks* like it queues but doesn't.

## Detection

```bash
# Find Dispatchable classes missing ShouldQueue
for f in app/Jobs/*.php app/Jobs/**/*.php; do
  if grep -q 'use Dispatchable' "$f" && ! grep -q 'implements ShouldQueue\|implements .* ShouldQueue' "$f"; then
    echo "MISSING ShouldQueue: $f"
  fi
done

# PHPStan rule (custom): require classes in app/Jobs/ to implement ShouldQueue
# (file a custom rule in your phpstan.neon ignoreErrors → become an architectural constraint)
```

Reference: [Laravel 13 — Queues: Creating Jobs](https://laravel.com/docs/13.x/queues#creating-jobs)
