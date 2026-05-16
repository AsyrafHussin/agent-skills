---
title: Skipped and Disabled Tests
impact: HIGH
impactDescription: "Dark coverage — code looks tested, isn't"
tags: testing, skip, disabled
---

## Skipped and Disabled Tests

**Impact: HIGH (Dark coverage — code looks tested, isn't)**

Skipped tests are worse than missing tests because they create a false sense of safety. A `test.skip(...)` or `markTestSkipped()` left in main without an owner, issue, or deadline is debt that grows in silence.

## How to Detect

```bash
# JavaScript / TypeScript (Jest / Vitest)
grep -rEn '\\.skip|xdescribe|xit|test\\.todo|describe\\.skip' tests/ src/

# Pest / PHPUnit
grep -rEn 'markTestSkipped|markTestIncomplete|@group\\s+skip|->skip\\(' tests/
```

Cross-reference each hit with:
- Is there a linked issue?
- Is there a comment explaining why?
- Is there a date or condition for re-enabling?

## Incorrect

```typescript
// ❌ Bare skips with no context
describe.skip('checkout', () => { /* ... */ });

test.skip('refunds work', () => { /* ... */ });

test('payment webhook', () => {
  if (process.env.CI) return;          // silent skip on CI
  // ...
});
```

**Problems:**
- Why are checkouts skipped? Nobody remembers
- The webhook test runs only locally — production behaviour is untested
- Coverage report shows them as "executed" but with zero assertions

## Correct

```typescript
// ✅ Every skip has an owner, reason, and re-enable trigger
test.skip(
  'refunds work — DISABLED 2026-02-10 (#1247) re-enable after Stripe webhook v2 migration',
  () => { /* ... */ }
);

// ✅ Or: delete and replace if the test cannot be repaired
//   git log will remember it ever existed.
```

Add CI checks:

```yaml
# .github/workflows/test-hygiene.yml
- name: Disallow new bare skips
  run: |
    NEW_SKIPS=$(git diff origin/main...HEAD -- 'tests/**' \
      | grep -E '^\+.*\.skip\(' | grep -v '#[0-9]')
    test -z "$NEW_SKIPS" || { echo "Bare skip without ticket"; exit 1; }
```

**Benefits:**
- Every skip is auditable and assigned
- New skips require a ticket — prevents quiet accumulation
- The team has a count of "real" coverage

## Remediation Strategy

- **Effort:** S per skip (decide: fix, delete, or document)
- **When to pay down:**
  - **Now:** audit existing skips → add owner + ticket OR delete
  - **Ongoing:** CI gate prevents new bare skips
- **Default action:** if a skip is older than 90 days with no movement, delete the test. If it's worth keeping, it's worth re-enabling.

Reference: [PHPUnit docs](https://docs.phpunit.de/) (see the "Incomplete and Skipped Tests" chapter in the current major version)
