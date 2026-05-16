---
title: "Doesn't Throw" Tests
impact: HIGH
impactDescription: "Tests that assert nothing meaningful; pass even when the function does the wrong thing"
tags: testing, weak-assertions, ai-fingerprint
---

## "Doesn't Throw" Tests

**Impact: HIGH (Tests that assert nothing meaningful; pass even when the function does the wrong thing)**

A test that calls the function and then asserts the function didn't throw is barely a test. It verifies one of the weakest possible properties — "the program didn't crash" — and gives false confidence in coverage. AI often defaults to this pattern because it's the easiest way to make a test pass.

A real test verifies a specific outcome. "Didn't throw" is an outcome only in the narrowest cases (you specifically want to verify a thrown error from a previous bug is now absent).

## Incorrect

```typescript
// ❌ Tests that just verify "the call completes"

describe('OrderService', () => {
  it('places an order', async () => {
    const service = new OrderService();
    await expect(service.place(makeValidOrder())).resolves.not.toThrow();
  });

  it('exports users', async () => {
    const service = new UserExportService();
    const result = await service.export();
    expect(result).toBeDefined();              // weak: undefined is the only failure mode caught
  });

  it('handles empty input', () => {
    const result = sum([]);
    expect(result).not.toBeNull();             // passes for 0, NaN, '', false, anything except null/undefined
  });
});
```

```php
// ❌ Same pattern in PHPUnit / Pest
test('it places an order', function () {
    $service = new OrderService();

    expect(fn () => $service->place(makeValidOrder()))->not->toThrow();
});

test('export returns something', function () {
    $result = (new UserExportService())->export();
    expect($result)->not->toBeNull();
});
```

**Why it's slop:**
- "Doesn't throw" tells you nothing about whether the right thing happened
- `place(order)` could return without throwing AND not actually place the order
- `export()` could return `undefined` (the test fails) OR `null` (test fails) OR an empty array, a misleading "true", etc. — the test catches only the narrowest failure
- `not.toBeNull()` passes for `0`, `''`, `NaN`, `false` — all of which are usually bugs

## Correct — assert what should happen

```typescript
// ✅ Specific outcomes

describe('OrderService', () => {
  it('place(order) persists the order and returns the saved row with status=pending', async () => {
    const service = new OrderService(db);
    const order = makeValidOrder({ total: 100_00 });

    const saved = await service.place(order);

    expect(saved.id).toBeDefined();
    expect(saved.status).toBe('pending');
    expect(saved.total).toBe(100_00);
    expect(await db.orders.findById(saved.id)).toMatchObject({ status: 'pending' });
  });

  it('exports all active users to CSV', async () => {
    await db.users.insertMany([
      { email: 'a@b.com', active: true },
      { email: 'c@d.com', active: false },
      { email: 'e@f.com', active: true },
    ]);

    const csv = await new UserExportService(db).export();

    expect(csv).toContain('a@b.com');
    expect(csv).toContain('e@f.com');
    expect(csv).not.toContain('c@d.com');     // inactive should be excluded
  });

  it('sum([]) returns 0', () => {
    expect(sum([])).toBe(0);
  });
});
```

**Why it reads human:**
- Each test verifies a specific, named outcome
- Bugs are detected: changing `status: 'pending'` to `'paid'` would fail the test; returning all users (not just active) would fail; `sum([])` returning `NaN` would fail
- Test names describe the *behaviour*, not the *function called*

## When "doesn't throw" IS legitimate

Rarely:
- **Smoke tests** for a never-throws contract on a public API (one or two tests; not the bulk of the suite)
- **Regression tests** for a specific previously-thrown error: "after fix #123, calling X with Y no longer throws"

In these cases, name the test clearly: `it('does not throw on empty input — regression for #123')`.

## Detection

```bash
# Tests whose only assertion is "not.toThrow" or "not.toBeNull"
grep -rEn '\.not\.toThrow\(|\.not\.toBeNull\(|\.toBeDefined\(' \
  --include='*.test.ts' --include='*.spec.ts' --include='*.test.tsx' src/ tests/ \
  | head -30

# PHPUnit / Pest equivalents
grep -rEn '\bexpect\(.*\)->not->toThrow\(\)|->assertNotNull\(\$result\)' \
  --include='*Test.php' --include='*.test.php' tests/

# Heuristic: test files where the ratio of weak assertions to strong assertions is high
# (manual review of files with > 5 weak assertions)
```

A few weak assertions are fine. A test suite dominated by them is a slop fingerprint.

Reference: [Martin Fowler — Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html) · Internal: [`test-mock-everything`](test-mock-everything.md), [`test-mirror-implementation`](test-mirror-implementation.md)
