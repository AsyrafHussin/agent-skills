---
title: '@deprecated Markers Without Removal Plan'
impact: MEDIUM
impactDescription: "Indefinite deprecations become permanent — they never actually go away"
tags: deprecated, lifecycle, process
---

## @deprecated Markers Without Removal Plan

**Impact: MEDIUM (Indefinite deprecations become permanent — they never actually go away)**

A `@deprecated` tag without a replacement, removal date, or migration plan is just a polite "I wish you wouldn't use this." Consumers keep using it, the deprecation message persists for years, and the codebase carries dead-but-not-dead APIs forever.

## How to Detect

```bash
# Find all @deprecated markers (one --include per extension; grep doesn't expand braces)
grep -rEn '@deprecated' \
  --include='*.php' --include='*.ts' --include='*.tsx' \
  --include='*.js' --include='*.jsx' src/ app/

# For each, check whether it includes:
#   - Replacement / `@see` pointer
#   - Removal version or date
#   - Reason

# Find @deprecated callers — these are migration targets
grep -rEn '@deprecated' -A2 src/ | grep -oE 'function [a-zA-Z]+|method [a-zA-Z]+'
```

Flag any `@deprecated` without:
1. **What to use instead** (`@see` or "use X instead")
2. **When it will be removed** (version, date, or trigger)
3. **At least one PR removing internal usage** since the deprecation was added

## Incorrect

```typescript
// ❌ Vague, unactionable deprecations
/** @deprecated */
export function getUserById(id: string) { /* ... */ }

/** @deprecated do not use */
export function legacyFormat(date: Date) { /* ... */ }

/** @deprecated use the new API */               // which "new API"?
export function fetchOrders() { /* ... */ }
```

**Problems:**
- Consumers see "deprecated" but cannot act
- No deadline → no urgency → no migration
- 5 years later still in code, still warning, still used everywhere

## Correct

```typescript
/**
 * @deprecated Since v4.2 (2025-09). Use {@link findUserById} which supports
 * batching and returns `Result<User, NotFound>`. Will be removed in v5.0
 * (target: 2026-Q3). Internal callers: 0 (migration complete).
 *
 * @see findUserById
 */
export function getUserById(id: string): User | null { /* ... */ }
```

Add CI to enforce:

```yaml
- name: No undocumented @deprecated
  run: |
    BAD=$(grep -rEn '@deprecated\b' src/ | grep -v -E '@deprecated.*[0-9]{4}')
    test -z "$BAD" || { echo "@deprecated needs a date/version"; exit 1; }
```

**Benefits:**
- Consumers see exactly what to do and when
- Tracking the deprecation → removal cycle is mechanical (grep + count)
- Old code actually leaves the codebase

## Remediation Strategy

- **Effort:** S per marker (audit), M per deprecation cycle (migrate internal callers, then remove)
- **When to pay down:**
  - **Audit existing markers:** add date + replacement, OR upgrade to "will be removed in next major" + start migration
  - **Removal:** when a deprecation hits its target version, *actually remove the code*. A deprecation that doesn't end is a lie.
- **Cycle:**
  1. Mark `@deprecated since vX (date), use Y, removed in vZ`
  2. Migrate internal callers (creates the proof the replacement works)
  3. Wait one major or N months for external callers
  4. Remove

Reference: [Semantic Versioning — Major changes](https://semver.org/) · [PHPDoc @deprecated](https://docs.phpdoc.org/3.0/guide/references/phpdoc/tags/deprecated.html)
