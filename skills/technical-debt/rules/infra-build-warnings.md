---
title: Ignored Build and Lint Warnings
impact: MEDIUM
impactDescription: "Warning noise hides real failures and trains the team to ignore output"
tags: build, lint, warnings, ci
---

## Ignored Build and Lint Warnings

**Impact: MEDIUM (Warning noise hides real failures and trains the team to ignore output)**

A build that emits dozens of warnings teaches every engineer that warnings are normal. The day a critical warning appears (a deprecation, a type-narrowing issue, a circular import), nobody sees it. Clean output is a precondition for noticing problems.

## How to Detect

```bash
# Capture and count warnings from build/test/lint
npm run build 2>&1 | grep -ciE 'warning|deprecat'
npx tsc --noEmit 2>&1 | wc -l
npx eslint . 2>&1 | grep -c 'warning'

# PHP
vendor/bin/phpstan analyse --no-progress
vendor/bin/phpcs --report=summary

# Webpack / Vite
# Look at the bundler output for "compiled with N warnings"
```

Threshold: **zero warnings tolerated**. Either fix or explicitly suppress with a comment explaining why.

## Incorrect

```bash
# ❌ Build "passes" but emits a wall of warnings
$ npm run build
[tsc] src/orders/index.ts(42,5): warning TS6133: 'unused' is declared but never used.
[tsc] src/orders/index.ts(55,3): warning TS2532: Object is possibly undefined.
[eslint] src/payment/stripe.ts:18:1 warning  no-explicit-any
[eslint] src/payment/stripe.ts:34:5 warning  react-hooks/exhaustive-deps
[webpack] WARNING in ./node_modules/some-pkg/dist/index.js
       Critical dependency: the request of a dependency is an expression
... 87 more warnings
Compiled with 92 warnings.
```

**Problems:**
- A new genuine warning ("X will be removed in vNext") buries in the noise
- "Compiled successfully" with 92 warnings is a lie that erodes trust
- New engineers conclude "warnings don't matter here"

## Correct

```bash
$ npm run build
Compiled successfully (0 warnings).
```

CI gates:

```yaml
- run: npx tsc --noEmit                          # fails on any type error
- run: npx eslint . --max-warnings 0             # zero warnings
- run: npm run build -- --no-warnings            # bundler warnings → errors
```

When a warning genuinely must be suppressed:

```typescript
// ✅ Targeted suppression with reason
// eslint-disable-next-line @typescript-eslint/no-explicit-any -- Stripe SDK types are too narrow; tracked in #2034
function configureStripe(opts: any) { /* ... */ }
```

**Benefits:**
- Build output is signal — every line means something
- Reviews can ask "does this PR add any new warning?" → easy answer
- Genuine deprecations and CVE-related warnings are noticed immediately

## Remediation Strategy

- **Effort:** S–M (most warnings are mechanical fixes; a few require small refactors)
- **When to pay down:**
  1. **Snapshot the current count** in CI: `--max-warnings $CURRENT_COUNT`
  2. **Ratchet down** — every PR can only equal or decrease the count
  3. Reach zero, then flip to `--max-warnings 0`
- **Anti-pattern:** disabling the lint rule entirely instead of fixing the underlying issues. The rule exists for a reason; suppress with context, don't disable globally.

Reference: [ESLint — Disabling Rules](https://eslint.org/docs/latest/use/configure/rules#disabling-rules) · [TypeScript Strict Mode](https://www.typescriptlang.org/tsconfig#strict)
