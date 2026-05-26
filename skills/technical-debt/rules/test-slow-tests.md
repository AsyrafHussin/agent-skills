---
title: Slow Test Suites
impact: HIGH
impactDescription: "Slow feedback compounds across every engineer every day"
tags: testing, performance, ci
---

## Slow Test Suites

**Impact: HIGH (Slow feedback compounds across every engineer every day)**

A test suite that takes 20 minutes costs every engineer 20 minutes per push. Multiplied across team size and PR count, it becomes the single biggest tax on velocity — and pushes the team to stop running tests locally.

## How to Detect

```bash
# Show slowest tests
npx jest --verbose                                    # per-test timing in output
npx jest --reporters=jest-slow-test-reporter          # 3rd-party reporter (recommended)
npx vitest --reporter=verbose | sort -k4 -rn | head -20

# PHPUnit
vendor/bin/phpunit --log-junit=junit.xml
# Then sort junit.xml by time

# Track total wall-clock time per CI run
# - Watch for trends: "is it growing 10% per quarter?"
```

Targets to aim for:
- **Unit suite:** < 30 seconds
- **Integration suite:** < 5 minutes
- **Total CI per PR:** < 10 minutes

## Incorrect

```typescript
// ❌ Common slow-test smells

// 1. Real network calls in unit tests
test('user can fetch profile', async () => {
  const data = await fetch('https://api.production.example.com/users/1');
});

// 2. Real sleeps for "wait for something"
test('debounced search', async () => {
  search('foo');
  await sleep(2000);          // 2s × 50 tests = 100s wasted
});

// 3. Full DB rebuild per test instead of transaction rollback
beforeEach(async () => {
  await execSync('npm run db:migrate:fresh');   // 10s × 200 tests
});
```

## Correct

```typescript
// ✅ Fast alternatives
// 1. Mock the network at the boundary for unit tests (MSW v2)
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';
const server = setupServer(
  http.get('/users/1', () => HttpResponse.json(USER)),
);

// 2. Wait on conditions, not on time
await waitFor(() => expect(results).toHaveLength(5));   // returns in ms

// 3. Use transactions or schema snapshots
beforeEach(async () => {
  await db.beginTransaction();
});
afterEach(async () => {
  await db.rollback();
});
```

Parallelize where safe:

```bash
# Jest: --maxWorkers=50%
# Vitest: vitest --pool=threads        (Vitest v1+; `--threads` is deprecated)
# PHPUnit: vendor/bin/paratest -p 8
```

**Benefits:**
- Engineers run tests locally → faster feedback before push
- CI cost drops linearly with wall time
- Slow-by-design tests (E2E) can be quarantined to a nightly job

## Remediation Strategy

- **Effort:** S–M per hotspot (mostly mechanical: mock, fake, paratest, transactionalize)
- **When to pay down:**
  1. **First win:** identify and fix the 5 slowest tests — usually 50%+ of total time
  2. **Then:** enable parallel execution
  3. **Then:** budget. Document a per-suite wall-clock budget and fail CI if exceeded.

**Budget enforcement:**

```yaml
# Hard cap test duration in CI
- run: timeout 600 npm test    # fails CI if > 10 minutes
```

Reference: [Martin Fowler — Test Suite Speed](https://martinfowler.com/articles/practical-test-pyramid.html#TheImportanceOf(Test)Speed)
