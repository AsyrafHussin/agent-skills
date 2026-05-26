---
title: Magic Numbers and Hardcoded Literals
impact: MEDIUM
impactDescription: "Obscure intent; require coordinated edits across files when changed"
tags: magic-numbers, constants, readability
---

## Magic Numbers and Hardcoded Literals

**Impact: MEDIUM (Obscure intent; require coordinated edits across files when changed)**

A `0.06` in tax code is invisible business knowledge. The next time the tax rate changes — or the next reader who needs to understand the rule — pays the cost. Magic numbers also make the same value drift across copies (one file uses `0.06`, another `0.065`).

## How to Detect

```bash
# TypeScript / JavaScript
npx eslint . --rule 'no-magic-numbers: ["error", { "ignore": [0, 1, -1] }]'

# PHP — PHPMD has no built-in magic-number rule; use a Psalm/PHPStan extension
# or a custom PHPCS sniff. Closest built-ins:
vendor/bin/phpstan analyse --level=8                       # catches some via type-aware analysis
# Custom: a project-local PHPCS sniff for hardcoded literals in *Service* / *Calculator* classes

# Cross-language grep for suspicious literals in business logic
grep -rEn '\b[0-9]+\.[0-9]+\b' app/Services/ src/services/ | grep -v test
```

Threshold: any non-trivial literal (anything except 0, 1, -1, and indexes used for slicing) appearing in business logic — especially if it appears more than once.

## Incorrect

```typescript
// ❌ Bare numbers and strings scattered through business logic
export function calculateOrder(items: Item[], user: User): Order {
  const subtotal = items.reduce((s, i) => s + i.price * i.qty, 0);
  const tax = subtotal * 0.06;                          // what's 0.06?
  const shipping = subtotal > 100 ? 0 : 15;             // why 100? why 15?
  const cacheKey = `order:${user.id}:v3`;               // why v3?
  redis.set(cacheKey, JSON.stringify({ subtotal, tax, shipping }), 'EX', 3600);  // 3600 what?
  return { subtotal, tax, shipping };
}
```

**Problems:**
- "0.06" appears in 4 other files — tax rate change requires hunting them all
- A reader can't tell whether `3600` is seconds, milliseconds, or a row count
- `'v3'` is silent invariant — changing cache format requires knowing about every caller

## Correct

```typescript
// ✅ Named constants with units and intent
const TAX_RATE = 0.06;
const FREE_SHIPPING_THRESHOLD = 100;
const FLAT_SHIPPING_FEE = 15;
const CACHE_VERSION = 'v3';
const CACHE_TTL_SECONDS = 60 * 60;

export function calculateOrder(items: Item[], user: User): Order {
  const subtotal = items.reduce((s, i) => s + i.price * i.qty, 0);
  const tax = subtotal * TAX_RATE;
  const shipping = subtotal > FREE_SHIPPING_THRESHOLD ? 0 : FLAT_SHIPPING_FEE;
  const cacheKey = `order:${user.id}:${CACHE_VERSION}`;
  redis.set(cacheKey, JSON.stringify({ subtotal, tax, shipping }), 'EX', CACHE_TTL_SECONDS);
  return { subtotal, tax, shipping };
}
```

**Benefits:**
- Tax-rate change is one line
- Units are explicit (`60 * 60` reads as "seconds in an hour")
- Constants are searchable; renames are mechanical

## Remediation Strategy

- **Effort:** S per file (extract constant, replace references)
- **When to pay down:** When you next change a value (the change is now one line), or when you spot the same literal in 2+ places.
- **Where to put constants:** as module-level `const` for local values; in a shared `pricing/Config.ts` or `config/billing.php` for cross-module business values.

**Tip:** for genuinely tunable values (rates, thresholds, feature flags), put them in environment-driven config so changes don't require a deploy.

Reference: [Refactoring — Replace Magic Number with Symbolic Constant](https://refactoring.guru/replace-magic-number-with-symbolic-constant)
