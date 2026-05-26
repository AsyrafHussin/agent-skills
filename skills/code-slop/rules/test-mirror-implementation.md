---
title: Tests That Mirror the Implementation
impact: HIGH
impactDescription: "Tests re-encode the production code; pass because they re-implement the same logic, not because they verify behaviour"
tags: testing, mirror-tests, ai-fingerprint
---

## Tests That Mirror the Implementation

**Impact: HIGH (Tests re-encode the production code; pass because they re-implement the same logic, not because they verify behaviour)**

AI generates tests by reading the function and translating its logic into the test. If `calculateTax` does `subtotal * 0.06`, the test sets `subtotal = 100`, then says "expect result to be `100 * 0.06`". The test passes because both sides compute the same thing. It would still pass if the function silently switched to `subtotal * 0.07` — IF the AI also "updated" the test to match.

Real tests state the **expected concrete answer**, not a formula computed from the input. They verify what *should* be true, not "the function does what the function does".

## Incorrect

```typescript
// ❌ The test re-implements the function

// Production:
export function calculateTax(subtotal: number): number {
  return subtotal * 0.06;
}

// Test:
import { calculateTax } from './calculateTax';

describe('calculateTax', () => {
  it('calculates tax', () => {
    const subtotal = 100;
    expect(calculateTax(subtotal)).toBe(subtotal * 0.06);    // re-encodes the formula
  });

  it('handles zero', () => {
    expect(calculateTax(0)).toBe(0 * 0.06);                  // tautology
  });
});
```

```php
// ❌ Same pattern in PHP
public function test_it_calculates_tax(): void
{
    $subtotal = 100;
    $expected = $subtotal * 0.06;                            // re-encoded
    $this->assertEquals($expected, calculateTax($subtotal));
}
```

**Why it's slop:**
- The test passes because RHS and LHS use the same formula — you've verified `x === x`
- If the formula in production silently changes to `0.07`, but the test "uses the formula" (or AI updates the test to match), the test still passes — and the bug ships
- Both sides of the equation come from the same place, so the test catches nothing the type system doesn't

## Correct — concrete, named expected values

```typescript
// ✅ Concrete expectations
describe('calculateTax', () => {
  it('charges 6% on the subtotal', () => {
    expect(calculateTax(100)).toBe(6);
    expect(calculateTax(50)).toBe(3);
    expect(calculateTax(1.50)).toBe(0.09);
  });

  it('returns 0 for a 0 subtotal', () => {
    expect(calculateTax(0)).toBe(0);
  });

  it('rejects negative subtotals', () => {
    expect(() => calculateTax(-10)).toThrow(InvalidSubtotal);
  });
});
```

```php
// ✅ Concrete expectations
public function test_charges_six_percent_on_subtotal(): void
{
    $this->assertEquals(6.0, calculateTax(100));
    $this->assertEquals(3.0, calculateTax(50));
    $this->assertEquals(0.09, calculateTax(1.50));
}
```

**Why it reads human:**
- The expected value is *named* — `6` is what 6% of 100 is, written as the answer
- If the production formula changes to `0.07`, the test fails at `expect(calculateTax(100)).toBe(6)` because `7 !== 6`
- The bug is caught at the boundary; the test holds the spec

## A particularly dangerous variant: bug-then-regenerate

```
User: "There's a bug — calculateTax is returning the wrong value for negatives."
AI: "I'll regenerate the test for you."
```

The regenerated test happens to pass for the new buggy behaviour. The test now ratifies the bug. This pattern is documented in Larridin (2025) and is one of the most insidious failure modes.

**Rule:** when fixing a bug, the test is written FIRST (red), with concrete expected values that reflect the correct behaviour. Then the production code is changed. Then the test goes green. Never accept "regenerate the test to match the new implementation".

## Detection

This is hard to detect mechanically — the pattern is "RHS includes the same constants/operations as the production code". Useful heuristics:

```bash
# Tests where expected values use the same magic number as the source
# (extract numeric constants from src/, then grep tests for them)
grep -rEoh '[0-9]+\.[0-9]+|[0-9]+_[0-9]+' --include='*.ts' src/ | sort -u > /tmp/src-constants.txt
grep -rEoh '[0-9]+\.[0-9]+|[0-9]+_[0-9]+' --include='*.test.ts' tests/ | sort -u > /tmp/test-constants.txt
comm -12 /tmp/src-constants.txt /tmp/test-constants.txt | head
# If many magic numbers from src appear in tests, they may be re-encoding the implementation

# Mutation testing is the gold-standard detection:
# - Stryker (JS/TS): https://stryker-mutator.io/
# - Infection (PHP): https://infection.github.io/
# If a mutation in the source doesn't fail a test, the test is mirror-ware.
```

**Mutation testing is the right answer here.** Add Stryker or Infection to your CI and require a minimum mutation score on PRs touching the unit-test directory.

Reference: [Stryker Mutator](https://stryker-mutator.io/) · [Infection](https://infection.github.io/) · Internal: [`test-mock-everything`](test-mock-everything.md), [`test-doesnt-throw`](test-doesnt-throw.md)
