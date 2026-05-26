---
title: Dead Code
impact: HIGH
impactDescription: "Unused code misleads readers and inflates maintenance surface"
tags: dead-code, unused, cleanup
---

## Dead Code

**Impact: HIGH (Unused code misleads readers and inflates maintenance surface)**

Dead code — unused exports, unreachable branches, commented-out blocks — costs nothing to delete and costs a lot to keep. Readers assume code that exists is code that runs; dead code wastes attention and creates phantom dependencies that block upgrades.

## How to Detect

```bash
# TypeScript / JavaScript
npx knip                     # unused files, exports, deps (preferred; ts-prune is archived)
npx eslint . --rule 'no-unreachable: error'

# PHP
vendor/bin/phpstan analyse --level=9  # detects unreachable code and unused private elements
#   For broader dead-code detection, add: tomasvotruba/unused-public, or use Rector's DeadCodeSetList

# Commented-out code (one --include per extension; grep doesn't expand braces)
grep -rEn '^\s*//.*[;{}]$' --include='*.php' --include='*.ts' --include='*.tsx' --include='*.js' .
```

## Incorrect

```typescript
// ❌ Dead imports, dead helper, dead branch, commented-out block
import { legacyFormatter } from './legacy';   // never used after v2 rewrite
import { format } from './format';

function formatPrice(p: number, currency: string) {
  // const oldImpl = (p) => `$${p.toFixed(2)}`;     // kept "just in case"
  // if (currency === 'BTC') return formatBtc(p);  // BTC support removed 2023

  if (currency === 'USD') return format(p, 'USD');
  if (currency === 'EUR') return format(p, 'EUR');
  return format(p, 'USD');
  return formatLegacy(p);   // unreachable
}

export function formatLegacy() { /* called nowhere */ }
```

**Problems:**
- Reader has to puzzle out whether the commented BTC branch is coming back
- `formatLegacy` blocks deleting the `./legacy` module
- The unreachable `return` raises false suspicion during reviews

## Correct

```typescript
// ✅ Delete it. Git remembers.
import { format } from './format';

function formatPrice(p: number, currency: string): string {
  if (currency === 'EUR') return format(p, 'EUR');
  return format(p, 'USD');
}
```

**Benefits:**
- No phantom dependency on the legacy module
- Reader sees only what runs
- Diff in `git log` documents *when* and *why* BTC was removed — better than a stale comment

## Remediation Strategy

- **Effort:** S (deletion is mechanical; trust git history)
- **When to pay down:** Immediately on detection — there is no reason to keep dead code in main.

**Note:** Resist the urge to keep "might-be-useful-later" code commented out. If you genuinely need it later, restore it from git history. The cost of a `git revert` is far less than the cost of confusing every future reader.

Reference: [Refactoring — Remove Dead Code](https://refactoring.guru/smells/dead-code)
