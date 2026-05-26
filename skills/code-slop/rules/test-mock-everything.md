---
title: Mock-Everything Tests That Assert Nothing
impact: CRITICAL
impactDescription: "Tests that pass forever; they re-encode the implementation rather than verify behaviour"
tags: testing, mocks, ai-fingerprint
---

## Mock-Everything Tests That Assert Nothing

**Impact: CRITICAL (Tests that pass forever; they re-encode the implementation rather than verify behaviour)**

When AI generates tests, the most common failure mode is "mock every dependency, then assert that the mocks were called". The test passes the moment it's written and passes forever — including when the underlying behaviour is silently broken. arXiv 2602.00409 (2026): coding agents produce significantly more over-mocked tests than human authors.

A real test verifies **outcome**, not **interaction**. Mocking a database to assert "create() was called with X" verifies that you wrote the implementation that calls `create()`. It doesn't verify that the customer record actually lands in the table.

## Incorrect

```typescript
// ❌ Mock-everything test that asserts mock interactions

import { describe, it, expect, vi } from 'vitest';
import { UserService } from './UserService';

describe('UserService', () => {
  it('createUser saves the user', async () => {
    const mockRepo = {
      save: vi.fn().mockResolvedValue({ id: 1, email: 'a@b.com' }),
      findByEmail: vi.fn().mockResolvedValue(null),
    };
    const mockMailer = { send: vi.fn() };
    const mockHasher = { hash: vi.fn().mockReturnValue('hashed') };

    const service = new UserService(mockRepo, mockMailer, mockHasher);
    await service.createUser({ email: 'a@b.com', password: 'x' });

    expect(mockRepo.save).toHaveBeenCalled();         // weak — passes if .save() called with anything
    expect(mockMailer.send).toHaveBeenCalled();
    expect(mockHasher.hash).toHaveBeenCalledWith('x'); // verifies you wrote .hash() — not that it's secure
  });
});
```

```php
// ❌ Same pattern in PHPUnit / Pest
test('processOrder calls payment gateway', function () {
    $payments = $this->mock(PaymentGateway::class);
    $payments->shouldReceive('charge')->once();      // verifies an interaction, not an outcome

    $service = new OrderService($payments);
    $service->process(new Order(/* ... */));
});
```

**Why it's slop:**
- The test passes even if `save()` stores the wrong fields, or `hash()` returns the input unchanged, or `charge()` skips actual payment
- It locks in the implementation's structure (every refactor breaks the test even when behaviour is fine)
- "Was the mock called" is rarely a useful assertion — what matters is "did the outcome happen"
- The test gives false confidence in coverage reports

## Correct — verify the outcome, with real dependencies where possible

```typescript
// ✅ Integration test against a real (in-memory) DB; assert what changed
import { describe, it, expect, beforeEach } from 'vitest';
import { createTestDb } from './testUtils';
import { UserService } from './UserService';

describe('UserService', () => {
  let db;
  beforeEach(async () => { db = await createTestDb(); });

  it('createUser creates a user row with hashed password and sends welcome email', async () => {
    const mailer = makeFakeMailer();   // collect-and-inspect, not a mock-with-asserts
    const service = new UserService(db.users, mailer);

    await service.createUser({ email: 'a@b.com', password: 'plaintext' });

    const stored = await db.users.findByEmail('a@b.com');
    expect(stored).toBeDefined();
    expect(stored!.email).toBe('a@b.com');
    expect(stored!.passwordHash).not.toBe('plaintext');     // hash actually happened
    expect(bcrypt.compareSync('plaintext', stored!.passwordHash)).toBe(true);
    expect(mailer.sentTo('a@b.com')).toHaveLength(1);       // outcome, not interaction
  });
});
```

```php
// ✅ Laravel: use the real database (RefreshDatabase) + fake the boundary services
test('processOrder charges customer and marks order paid', function () {
    Mail::fake();
    Http::fake([
        'api.stripe.com/*' => Http::response(['id' => 'ch_test_123', 'status' => 'succeeded'], 200),
    ]);

    $order = Order::factory()->create(['status' => 'pending', 'total' => 100_00]);

    (new OrderService(new StripeGateway()))->process($order);

    expect($order->fresh()->status)->toBe('paid');
    expect($order->fresh()->stripe_charge_id)->toBe('ch_test_123');
    Mail::assertSent(OrderConfirmation::class, fn ($m) => $m->order->is($order));
});
```

**Why it reads human:**
- Each assertion checks **what should be true after the operation** — not "did you call X"
- Refactoring the internal calls is safe — the test still passes if the outcomes hold
- Real bugs (forgotten hash, wrong field assignment, missing email) trigger the test failures
- The test reads like a customer story: "after createUser, the user exists, password is hashed, welcome email is sent"

## When mocks ARE appropriate

A few legitimate uses:

- **Cost / side-effect boundaries** — real Stripe calls (use `Http::fake()` in Laravel; mock the HTTP boundary)
- **Slow externals** — third-party APIs with rate limits (mock the HTTP boundary)
- **Time-sensitive code** — freeze the clock (Carbon test helpers, vi.setSystemTime), don't mock all of time
- **Hard-to-reproduce error paths** — force a `ConnectionException` to test the retry handler

The pattern: **mock at the network/IO boundary, not at every internal class.**

## Detection

```bash
# Tests that import the mock library more than the assertion library (rough)
grep -rEn '(vi\.fn|jest\.fn|->mock\(|->shouldReceive)' \
  --include='*.test.ts' --include='*.spec.ts' --include='*Test.php' \
  | wc -l

# Tests asserting only on mock interactions, no DB / outcome checks
# (heuristic: file has 'toHaveBeenCalled' / 'shouldReceive' but no 'expect(<repo>.find' or 'assertDatabaseHas')
for f in $(find . -name '*Test.php' -o -name '*.test.ts' 2>/dev/null); do
  has_mock=$(grep -cE 'toHaveBeenCalled|shouldReceive' "$f")
  has_outcome=$(grep -cE 'assertDatabaseHas|->fresh\(|findByEmail|toBe\(|toEqual\(' "$f")
  if [ "$has_mock" -gt 3 ] && [ "$has_outcome" -lt 1 ]; then
    echo "MOCK-ONLY: $f"
  fi
done
```

Reference: [arXiv 2602.00409 — Are Coding Agents Generating Over-Mocked Tests?](https://arxiv.org/abs/2602.00409) · Internal: [`test-mirror-implementation`](test-mirror-implementation.md), [`test-doesnt-throw`](test-doesnt-throw.md)
