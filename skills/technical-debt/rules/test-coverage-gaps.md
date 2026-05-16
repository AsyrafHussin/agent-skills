---
title: Coverage Gaps on Critical Paths
impact: HIGH
impactDescription: "Uncovered critical paths fail in production, not in CI"
tags: testing, coverage, regression
---

## Coverage Gaps on Critical Paths

**Impact: HIGH (Uncovered critical paths fail in production, not in CI)**

Aggregate coverage percentage is a vanity metric. What matters is whether the **critical paths** (checkout, auth, payment, signup) have integration tests. A repo at 85% line coverage with zero tests on payment has more risk than one at 60% with a full checkout suite.

## How to Detect

Identify critical paths from the product (signup, login, checkout, refund, etc.), then check:

```bash
# Node
npx jest --coverage --coverageReporters=text
npx vitest run --coverage

# PHP
vendor/bin/phpunit --coverage-html=coverage/

# Targeted: are there any integration tests for the checkout flow?
grep -rln 'test.*checkout\|describe.*checkout' tests/
```

For each critical path, look for:
- An end-to-end / integration test that exercises the happy path
- Tests for failure modes (declined card, out-of-stock, expired session)
- At least one test that runs against the real DB / real network adapter

## Incorrect

```
// ❌ 92% line coverage — but all of it is on getters, mappers, and trivial helpers.
// The actual checkout pipeline has NO integration test.

src/
├── utils/             — 100% covered (10 tests)
├── formatters/        — 100% covered (15 tests)
├── checkout/
│   ├── pricing.ts     —  20% covered
│   ├── inventory.ts   —   0% covered
│   ├── payment.ts     —   0% covered
│   └── orchestrate.ts —   0% covered   ← THE checkout flow
```

**Problems:**
- Coverage metric is "green" → team feels safe → bugs land in checkout
- No safety net for refactoring the orchestrator
- On-call has no automated regression check before deploys

## Correct

```typescript
// ✅ One integration test per critical-path scenario, hitting real adapters where feasible
// tests/checkout.integration.test.ts
describe('checkout', () => {
  it('places an order with valid card', async () => {
    const order = await checkoutClient.place({ /* ... */ });
    expect(order.status).toBe('confirmed');
    expect(inventory.stockFor('SKU-1')).toBe(initial - 1);
  });

  it('rejects when card is declined', async () => {
    await expect(checkoutClient.place({ token: 'tok_chargeDeclined' }))
      .rejects.toThrow(PaymentDeclined);
    expect(inventory.stockFor('SKU-1')).toBe(initial);  // no leak
  });

  it('rejects when item is out of stock', async () => { /* ... */ });
  it('issues idempotent retries safely', async () => { /* ... */ });
});
```

**Benefits:**
- Regression on checkout fails CI, not customers
- Refactoring the orchestrator is safe
- New scenarios (e.g., new payment method) extend a known suite

## Remediation Strategy

- **Effort:** M per critical path (the first test costs the most; subsequent are cheap)
- **When to pay down:** **Before** the next behaviour change on that path. The change itself is your justification.
- **Target:** one integration test per critical path, covering happy + 2–3 failure modes. Don't chase a coverage number.

Reference: [Martin Fowler — Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
