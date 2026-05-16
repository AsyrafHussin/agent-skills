---
title: Defensive in the Wrong Places — Missing the Real Defenses
impact: CRITICAL
impactDescription: "Code looks safe but the *actual* failure modes (network, queues, races) are unprotected"
tags: defensive-programming, timeouts, resilience, ai-fingerprint
---

## Defensive in the Wrong Places — Missing the Real Defenses

**Impact: CRITICAL (Code looks safe but the actual failure modes — network, queues, races — are unprotected)**

OX Security 2025: **76% of AI-assisted PRs miss timeouts on external calls**. The same PRs are full of impossible null checks and generic try/catch. The pattern is: defensive in places that don't need it; not defensive in places that do.

What AI often misses (the failure modes that actually take down production):

- **No timeouts on outbound HTTP** — a slow third-party API hangs your request indefinitely
- **No retry policy + jitter** — a transient blip cascades into a customer-visible failure
- **No idempotency keys on payment/order creation** — a network retry creates duplicate charges
- **No rate-limit checks before hitting an external API** — banned by Stripe / Shopify / etc.
- **No circuit breaker** — keeps hammering a known-down dependency, queue backs up
- **No locking / unique constraint** — race condition double-spends inventory
- **No backpressure on workers** — queue grows unbounded; OOM

These are the defenses that matter. A robust system has these; the dramatic-looking try/catch and null checks don't replace them.

## Incorrect

```php
// ❌ Defensive theatre: try/catch wraps the wrong thing; real risks unprotected

public function syncOrder(string $orderId): void
{
    try {
        // No timeout — could hang for minutes
        $response = Http::get("https://api.shopify.com/orders/{$orderId}");
        $data = $response->json();

        if ($data === null) {            // impossible — Http returns array or throws
            Log::error('Sync failed');
            return;
        }

        // No idempotency — if this retries, we create duplicates
        Order::create($data);
    } catch (Exception $e) {
        Log::error('Sync failed: ' . $e->getMessage());
    }
}
```

```typescript
// ❌ Same pattern in TS
async function syncOrder(orderId: string): Promise<void> {
  try {
    // No timeout; no retry; no backoff
    const res = await fetch(`https://api.shopify.com/orders/${orderId}`);
    const data = await res.json();

    if (!data) {       // impossible if res.json() resolved
      console.error('Sync failed');
      return;
    }

    await db.orders.create({ data });
  } catch (e) {
    console.error(e);    // swallow
  }
}
```

**Why it's slop:**
- The `if ($data === null)` is dead defence; the actual failure mode is "Shopify takes 90s to respond and our request timeout hits us first"
- The try/catch swallows real failures but doesn't add the things that prevent them
- Retried calls create duplicate orders (idempotency missing)
- Looks responsible; isn't

## Correct

```php
// ✅ The real defences — timeouts, idempotency, retry with backoff, circuit-break

public function syncOrder(string $orderId): void
{
    $response = Http::timeout(10)                                  // hard timeout
        ->retry(times: 3, sleepMilliseconds: 200, when: fn ($e) =>
            $e instanceof ConnectionException || $e->getCode() === 429)
        ->get("https://api.shopify.com/orders/{$orderId}");

    $response->throw();   // throw on 4xx/5xx — let it propagate

    Order::updateOrCreate(
        ['shopify_id' => $orderId],                                // idempotency via unique key
        $response->json()
    );
}
```

```typescript
// ✅ TS with AbortController for timeout + retry policy
async function syncOrder(orderId: string): Promise<void> {
  const data = await fetchWithRetry(
    `https://api.shopify.com/orders/${orderId}`,
    { timeoutMs: 10_000, retries: 3, retryOnStatus: [429, 502, 503, 504] },
  );

  // Idempotent upsert by external id (unique index on shopify_id)
  await db.orders.upsert({
    where: { shopify_id: orderId },
    create: data,
    update: data,
  });
}
```

**Why it reads human:**
- The actual failure modes (slow third party, transient errors, duplicate retries) are each addressed
- Errors NOT covered by retry propagate — observability/alerts catch them
- The idempotency key (`shopify_id`) prevents duplicate orders even under retry storms
- The "try/catch" is gone — it added no value here

## The real-defences checklist

For any code that talks to the outside world, ask:

- [ ] **Timeout** on every outbound call (HTTP, DB, queue, cache)?
- [ ] **Retry** policy: how many, with what backoff, on which error types?
- [ ] **Idempotency**: if the call is retried, will it create duplicate side effects?
- [ ] **Rate limit** awareness: am I tracking my own request rate or relying on the upstream's error?
- [ ] **Circuit breaker** for known-down dependencies (or at least a backoff cap)?
- [ ] **Locking / unique constraints** on writes that could race?
- [ ] **Backpressure** on workers reading from a queue?
- [ ] **Real errors propagated** to observability instead of swallowed?

A try/catch that doesn't add any of these isn't a defense. It's theatre.

## Detection

```bash
# HTTP calls without explicit timeout (Laravel Http facade)
grep -rEn 'Http::get\(|Http::post\(|Http::put\(|Http::delete\(' --include='*.php' app/ \
  | grep -v 'timeout('

# fetch() calls without AbortController / signal
grep -rEn 'await fetch\(' --include='*.ts' --include='*.tsx' --include='*.js' src/ \
  | grep -vE 'signal:|AbortController'

# create() / insert() without idempotency check (rough — manual review)
grep -rEn '(Order|Charge|Payment)::create\(' --include='*.php' app/
```

A repo can pass every other rule in this skill and still ship the wrong defenses. **This rule is the one to take seriously on payment, checkout, and integration code paths.**

Reference: [Stripe — Idempotent Requests](https://docs.stripe.com/api/idempotent_requests) · [Hystrix / Resilience4j circuit breaker patterns](https://github.com/resilience4j/resilience4j) · OX Security study · Internal: [`defensive-generic-catch`](defensive-generic-catch.md)
