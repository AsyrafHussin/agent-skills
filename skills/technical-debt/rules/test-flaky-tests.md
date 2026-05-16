---
title: Flaky Tests
impact: HIGH
impactDescription: "Erode trust in CI; teach the team to ignore failures"
tags: testing, flaky, ci
---

## Flaky Tests

**Impact: HIGH (Erode trust in CI; teach the team to ignore failures)**

A flaky test — one that passes and fails on the same code — is worse than no test. The first few times, you re-run. After that, the team learns to retry-and-merge, and the suite stops catching real regressions.

## How to Detect

```bash
# Run the suite N times against the same commit
for i in {1..20}; do npm test -- --silent || echo "Failed run $i"; done

# Better: track flakiness across CI runs
# - GitHub Actions: re-run-on-failure stats
# - CircleCI / Buildkite: built-in flaky-test reports
# - https://github.com/jonny-improbable/jest-circus-flaky-retry

# Identify suspect tests by name patterns
grep -rEn 'sleep|setTimeout|Date\.now|Math\.random' tests/
```

Common flaky-test smells:
- `sleep(N)` / `setTimeout` instead of waiting for a condition
- Order-dependent tests (depend on previous test's state)
- Time-dependent assertions (`expect(date).toBe(today)`)
- Tests against shared mutable resources (real Redis without cleanup, real DB without transactions)

## Incorrect

```typescript
// ❌ Three different forms of flakiness
test('debounced search', async () => {
  searchInput.value = 'foo';
  await sleep(300);                          // race: timing-dependent
  expect(results).toHaveLength(5);
});

test('order created today', () => {
  const order = createOrder();
  expect(order.createdAt.toDateString())
    .toBe(new Date().toDateString());        // fails when run at midnight
});

test('user can log in', async () => {
  await db.query('INSERT INTO users ...');  // depends on previous test's cleanup
  // ...
});
```

**Problems:**
- Sleep races: works on fast machines, fails on busy CI runners
- Date-dependent: fails on DST changes, midnight, leap-day
- Shared-state: passes alone, fails in suite

## Correct

```typescript
// ✅ Deterministic alternatives
test('debounced search', async () => {
  searchInput.value = 'foo';
  await waitFor(() => expect(results).toHaveLength(5));  // wait on condition
});

test('order created at the expected time', () => {
  vi.setSystemTime(new Date('2026-01-15T10:00:00Z'));    // freeze time
  const order = createOrder();
  expect(order.createdAt).toEqual(new Date('2026-01-15T10:00:00Z'));
});

beforeEach(async () => {
  await db.transaction(async (t) => { /* setup, rolled back after each test */ });
});
```

**Benefits:**
- Test passes deterministically regardless of machine speed, clock, or order
- Suite can run in parallel without contention
- CI failures become signal again

## Remediation Strategy

- **Effort:** S per test (the fix is usually local — replace sleep with waitFor, freeze the clock, use transactions)
- **When to pay down:**
  1. **Quarantine** the flaky test immediately (mark as `.skip` with a tracking issue) so it stops eroding trust
  2. **Fix** within the sprint — quarantine is a deferral, not a destination
  3. **Delete** if it can't be made deterministic in reasonable effort

**Policy:** Re-running a failed CI without a root-cause is anti-pattern. Always file an issue.

Reference: [Google Testing Blog — Flaky Tests](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html)
