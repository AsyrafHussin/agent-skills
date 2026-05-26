---
title: Generic catch Blocks That Don't Distinguish Errors
impact: CRITICAL
impactDescription: "82% of AI PRs (OX Security) — masks bugs as 'handled', breaks observability, kills debuggability"
tags: defensive-programming, error-handling, ai-fingerprint
---

## Generic catch Blocks That Don't Distinguish Errors

**Impact: CRITICAL (82% of AI PRs per OX Security study; masks bugs as 'handled', breaks observability, kills debuggability)**

The single most-cited AI anti-pattern in 2025 research. Try/catch wrapped around code, with a generic catch that logs and swallows. Looks responsible. Is the opposite — it hides bugs and converts loud failures into silent corruption.

Real error handling distinguishes:
- **What can throw** (specific exception types)
- **What's recoverable** (catch, retry, fall back)
- **What's a bug** (re-throw, let it propagate)

A bare `catch (e) { console.error(e); }` does none of this. It says "if anything goes wrong, I'll keep going" — and "anything" includes programmer errors that should crash so they're noticed.

## Incorrect

```php
// ❌ Generic catch that swallows everything

public function processPayment(Order $order): bool
{
    try {
        $charge = $this->stripe->charges->create([
            'amount'   => $order->total->cents(),
            'currency' => 'usd',
            'source'   => $order->paymentToken,
        ]);
        $order->update(['stripe_charge_id' => $charge->id, 'status' => 'paid']);
        return true;
    } catch (Exception $e) {
        Log::error('Payment failed: ' . $e->getMessage());
        return false;
    }
}
```

```typescript
// ❌ Same TS pattern
async function exportUsers(): Promise<User[]> {
  try {
    const users = await api.fetchAllUsers();
    return users;
  } catch (e) {
    console.error('Export failed:', e);
    return [];
  }
}
```

**Why it's slop:**
- `Exception $e` catches *everything*, including `TypeError` (programmer bug) and `OutOfMemoryError` (system crash) — those should not be "handled" by logging and continuing
- Returning `false` / `[]` lets the caller think the operation succeeded with no data, masking a real failure
- `Log::error` lacks context — what was the order? what user? what amount? Future debugger has no anchor
- A real Stripe error (card declined) gets the same treatment as a typo in the code

## Correct

```php
// ✅ Catch specifically; re-throw what you don't understand

public function processPayment(Order $order): void
{
    try {
        $charge = $this->stripe->charges->create([
            'amount'   => $order->total->cents(),
            'currency' => 'usd',
            'source'   => $order->paymentToken,
        ]);
        $order->update(['stripe_charge_id' => $charge->id, 'status' => 'paid']);
    } catch (CardException $e) {
        // Customer-facing failure — known, expected
        $order->update(['status' => 'declined', 'decline_reason' => $e->getStripeCode()]);
        throw new PaymentDeclined($e->getStripeCode(), previous: $e);
    } catch (RateLimitException $e) {
        // Transient — caller retries
        throw new TransientPaymentFailure(retryAfterSeconds: 30, previous: $e);
    }
    // Any other exception propagates: TypeError, OutOfMemoryError, etc.
    // — those are bugs we WANT to know about.
}
```

```typescript
// ✅ Specific errors, propagate the unknown
async function exportUsers(): Promise<User[]> {
  try {
    return await api.fetchAllUsers();
  } catch (e) {
    if (e instanceof AbortError) {
      // Caller aborted; that's a feature, not a failure
      throw e;
    }
    if (e instanceof RateLimitError) {
      await sleep(e.retryAfterMs);
      return api.fetchAllUsers();  // one retry
    }
    // Network / parse / bug — propagate; don't return [] (the caller would think it succeeded with no users)
    throw e;
  }
}
```

**Why it reads human:**
- Each `catch` branch handles a specific, named condition with a specific remediation
- Unknown errors propagate — the system fails loud, observability catches them, on-call wakes up before customers do
- No silent "return false" / "return empty array" — caller can't accidentally treat failure as empty success

## The "what would I want at 2am?" test

When debugging a production incident, you want:
- **Errors at the right loudness** — bugs crash with stack traces; expected failures have domain-specific exceptions you can grep
- **Context, not just the message** — order ID, user ID, request ID, all in the log line
- **No silent corruption** — a "successful" call that returned no data is the worst kind of failure

The generic-catch pattern fails all three.

## Detection

```bash
# Bare catches in PHP
grep -rEn 'catch\s*\(\s*\\?Exception\b' --include='*.php' app/

# Bare catches in TS / JS
grep -rEn '\}\s*catch\s*\(\s*[a-zA-Z_]+\s*\)\s*\{' --include='*.ts' --include='*.tsx' --include='*.js' src/

# Catch blocks that only log and continue
grep -rEnB1 'console\.error\(.*\)\s*;\s*$' --include='*.ts' --include='*.tsx' src/

# PHP equivalent
grep -rEnB1 'Log::(error|warning)\(.*\)' --include='*.php' app/Http/ app/Services/

# Linter rules
# ESLint: no-empty (already standard); add @typescript-eslint/no-explicit-any
# PHPStan level >= 8 flags catch(\Throwable) in some configurations
```

A small handful of generic catches is fine (e.g., at HTTP boundaries with structured logging). **Density** is the signal — > 5 generic catches in a single service is the AI fingerprint.

Reference: Internal: [`defensive-impossible-null`](defensive-impossible-null.md), [`defensive-missing-real`](defensive-missing-real.md)
