---
title: Cyclomatic and Cognitive Complexity
impact: CRITICAL
impactDescription: "High-complexity methods harbor most production bugs"
tags: complexity, cyclomatic, cognitive
---

## Cyclomatic and Cognitive Complexity

**Impact: CRITICAL (High-complexity methods harbor most production bugs)**

Methods with cyclomatic complexity > 10 (or cognitive complexity > 15) are statistically the strongest predictor of bug density in a codebase. They are also the slowest to test, review, and modify.

## How to Detect

```bash
# PHP
vendor/bin/phpmd app text codesize --reportfile complexity.txt

# JavaScript / TypeScript
npx eslint . --rule 'complexity: ["error", 10]'
```

Threshold: **cyclomatic complexity > 10** OR **cognitive complexity > 15** OR **nesting depth > 4**.

## Incorrect

```typescript
// ❌ Cyclomatic complexity 14 — every branch combination is a separate path
function calculateDiscount(user: User, order: Order): number {
  if (user.tier === 'gold') {
    if (order.total > 1000) {
      if (order.itemCount > 10) {
        if (user.yearsActive > 5) return 0.30;
        else return 0.25;
      } else if (order.itemCount > 5) {
        return 0.20;
      }
      return 0.15;
    } else if (order.total > 500) {
      return user.yearsActive > 2 ? 0.12 : 0.10;
    }
    return 0.05;
  } else if (user.tier === 'silver') {
    // ... another 20 lines of nested ifs
  }
  return 0;
}
```

**Problems:**
- 14+ independent code paths — minimum 14 tests to cover them all
- Adding a new tier requires understanding every nested branch
- Reviewers cannot hold the state in their head

## Correct

```typescript
// ✅ Replace nested conditions with a lookup table + small predicates
type DiscountRule = { match: (u: User, o: Order) => boolean; rate: number };

const DISCOUNT_RULES: DiscountRule[] = [
  { match: (u, o) => u.tier === 'gold' && o.total > 1000 && o.itemCount > 10 && u.yearsActive > 5, rate: 0.30 },
  { match: (u, o) => u.tier === 'gold' && o.total > 1000 && o.itemCount > 10,                       rate: 0.25 },
  { match: (u, o) => u.tier === 'gold' && o.total > 1000 && o.itemCount > 5,                        rate: 0.20 },
  // ...
];

function calculateDiscount(user: User, order: Order): number {
  return DISCOUNT_RULES.find(r => r.match(user, order))?.rate ?? 0;
}
```

**Benefits:**
- Each rule is independently testable
- New tiers/rules added by appending — no nested-branch surgery
- Cyclomatic complexity of `calculateDiscount` drops to 1

## Remediation Strategy

- **Effort:** S–M per method
- **When to pay down:** The next time you need to add a branch to a complexity-flagged method. Extract first, modify after.

Reference: [SonarSource — Cognitive Complexity](https://www.sonarsource.com/resources/cognitive-complexity/)
