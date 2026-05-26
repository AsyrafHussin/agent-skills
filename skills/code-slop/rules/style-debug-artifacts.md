---
title: Debug Artifacts Left in Production Code
impact: HIGH
impactDescription: "console.log, dd(), dump(), var_dump — AI's exploratory leftovers ship to production"
tags: style, debug, ai-fingerprint, cleanup
---

## Debug Artifacts Left in Production Code

**Impact: HIGH (console.log, dd(), dump(), var_dump — AI's exploratory leftovers ship to production)**

`console.log("here")`, `console.log("got user", user)`, `dd($order)`, `dump($result)`, `var_dump($payload)`, `print_r($data)`, `echo $error` — these are the breadcrumbs left from when the developer (or AI) was debugging. They ship to production and:

- Leak sensitive data into stdout / log aggregators (PII, tokens)
- Bloat production logs to the point you can't grep for real signal
- `dd()` literally halts execution — if it reaches prod, your endpoint returns "1" + var_dump output instead of JSON

AI is particularly bad about this because the model tends to add `console.log("got result", x)` "to help with debugging" and rarely removes it before "finalising" the function.

## Incorrect

```typescript
// ❌ Debug artifacts left in

async function processPayment(order: Order, token: string): Promise<Charge> {
  console.log('processPayment start', order.id);                  // shipped
  console.log('token', token);                                    // SHIPS THE TOKEN
  const charge = await stripe.charges.create({ /* ... */ });
  console.log('got charge', charge);                              // shipped
  return charge;
}
```

```php
// ❌ Same in PHP
public function processWebhook(Request $request): JsonResponse
{
    $payload = $request->json()->all();
    dd($payload);                                                 // halts execution; returns a debug page
    // …rest never runs
}

public function calculateTax(Order $order): Money
{
    dump($order);                                                 // prints to stdout in production
    print_r($order->items);
    $taxRate = 0.06;
    var_dump($taxRate);
    return $order->subtotal->multiplied($taxRate);
}
```

**Why it's slop:**
- `dd()` in a controller is an outage — the request never completes
- `console.log('token', token)` is a credentials leak; on serverless logs, every Stripe call ships the token to CloudWatch
- `dump()` / `var_dump()` show up in HTTP responses if not in a CLI context (especially during `artisan tinker` or test failures)
- A repo with 50+ stray `console.log` in production paths signals nobody is reading their own code before merging

## Correct

```typescript
// ✅ No debug; if logging matters, use the proper logger with context
import { logger } from '@/lib/logger';

async function processPayment(order: Order, token: string): Promise<Charge> {
  // Real logger — structured, redacts secrets, levels enforced
  const log = logger.child({ orderId: order.id });
  log.info('payment.start');

  const charge = await stripe.charges.create({ /* ... */ });

  log.info('payment.success', { chargeId: charge.id, amountCents: charge.amount });
  return charge;
}
```

```php
// ✅ Structured logging at the boundary; no debug() calls

public function processWebhook(Request $request): JsonResponse
{
    Log::withContext([
        'webhook_id' => $request->header('Stripe-Webhook-Id'),
        'event_type' => $request->json('type'),
    ])->info('webhook.received');

    // … actual handling …

    return response()->json(['ok' => true]);
}
```

**Why it reads human:**
- A logger with structured fields and levels (info/warn/error) — not stdout spam
- Tokens / secrets get redacted by the logger (or not logged at all)
- The log lines are intentional, useful for production debugging, and won't break the response

## When debug calls in production code ARE warranted

Rare. Usually zero. Specific cases:

- **Logs in well-defined CLI scripts** that are *meant* to be verbose: a one-off data migration script can use `echo` / `console.log` freely
- **Test-only files** (`*.test.ts`, `*Test.php`) — fine to keep debug there during development
- **Explicit `if (DEBUG_MODE) console.log(...)`** wrapped behind a feature flag — fine, but rare in practice

Production controllers, services, jobs, listeners, middleware: **zero raw debug calls**.

## Detection

```bash
# JavaScript / TypeScript — console.log in production code
grep -rEn '\bconsole\.(log|debug|info|warn)\(' --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' \
  src/ resources/js/ 2>/dev/null \
  | grep -v -E '\.test\.|\.spec\.|/__tests__/|/scripts/'

# PHP — debug helpers
grep -rEn '\b(dd|dump|var_dump|print_r|var_export)\s*\(' --include='*.php' \
  app/ 2>/dev/null

# CI gate — block PRs that introduce debug artifacts
NEW_DEBUG=$(git diff --diff-filter=ACM origin/main...HEAD -- 'app/**/*.php' 'src/**/*.ts' \
  | grep -E '^\+.*\b(console\.log|dd\(|dump\(|var_dump\(|print_r\()')
test -z "$NEW_DEBUG" || { echo "Debug artifacts in PR:"; echo "$NEW_DEBUG"; exit 1; }
```

ESLint rules:

```json
{
  "rules": {
    "no-console": ["error", { "allow": ["warn", "error"] }]
  }
}
```

PHPStan + a custom rule can flag `dd`/`dump` similarly. PHP-CS-Fixer has a `no_debug_print` rule.

Reference: [Laravel Logging docs](https://laravel.com/docs/logging) · [ESLint no-console](https://eslint.org/docs/latest/rules/no-console) · [Pino structured logging](https://github.com/pinojs/pino)
