---
title: Empty Docblocks That Restate the Signature
impact: HIGH
impactDescription: "Adds bytes, zero information; suggests the author auto-generated without reading"
tags: comments, docblocks, jsdoc, phpdoc, ai-fingerprint
---

## Empty Docblocks That Restate the Signature

**Impact: HIGH (Adds bytes, zero information; suggests the author auto-generated without reading)**

A docblock that adds nothing the signature already says is a tell. `/** Get the user */` above `function getUser(): User` tells the reader nothing the function name and return type don't. Modern PHP and TypeScript both encode types in the signature; redundant docblocks are pure noise.

Useful docblocks document **edge cases, thrown exceptions, side effects, or non-obvious semantics** — not "this method does what its name says".

## Incorrect

```php
// ❌ Restates the signature; adds nothing
/**
 * Get the user by ID.
 *
 * @param int $id The user ID
 * @return User The user
 */
public function getUser(int $id): User
{
    return User::findOrFail($id);
}

/**
 * Adds two numbers together.
 *
 * @param int $a First number
 * @param int $b Second number
 * @return int The sum
 */
public function add(int $a, int $b): int
{
    return $a + $b;
}
```

```typescript
// ❌ Same pattern in TS
/**
 * Get the user.
 * @param id - The user ID
 * @returns The user
 */
function getUser(id: string): User { /* ... */ }

/**
 * Adds two numbers.
 * @param a - First number
 * @param b - Second number
 * @returns The sum
 */
function add(a: number, b: number): number { return a + b; }
```

**Why it's slop:**
- The signature already says all of this — return type, parameter types, what's returned
- Bytes per useful info: ~zero
- A human writes a docblock when there's something the signature *can't* express

## Correct — option 1: no docblock at all

```php
// ✅ Signature speaks for itself
public function getUser(int $id): User
{
    return User::findOrFail($id);
}

public function add(int $a, int $b): int
{
    return $a + $b;
}
```

## Correct — option 2: docblock that earns its place

```php
// ✅ Docblock documents what the signature CAN'T say
/**
 * Get a user, or throw if not found.
 *
 * @throws ModelNotFoundException if no user matches $id
 * @throws AuthorizationException if the current user can't view $id
 */
public function getUser(int $id): User
{
    Gate::authorize('view', User::class);
    return User::findOrFail($id);
}
```

```typescript
// ✅ TSDoc that documents real edge cases
/**
 * @throws OutOfRangeError if `amount` exceeds the Stripe 999,999.99 cap.
 * @remarks Rounds to 2 decimal places using banker's rounding.
 */
function formatAmountForStripe(amount: number): string { /* ... */ }
```

**Why it reads human:**
- The remaining docblocks all carry information the signature can't
- A reader scanning the file sees docblocks and trusts they're load-bearing

## Detection

```bash
# Files with @param/@return docblocks
grep -rln '@param\|@return\|@returns' --include='*.php' app/
grep -rln '@param\|@returns' --include='*.ts' --include='*.tsx' src/

# PHPStan can flag redundant @param when the typed signature is more informative:
vendor/bin/phpstan analyse --level=9
# (set `reportAlwaysTrueInLastCondition: true` and use phpstan-deprecation-rules)

# ESLint rule for TS — flags JSDoc with no additional info:
# https://github.com/gajus/eslint-plugin-jsdoc → jsdoc/require-jsdoc { contexts: 'never' }
# Or simpler: turn OFF require-jsdoc and let humans write docblocks only when warranted.
```

The most reliable enforcement is a **PR review rule**: "every `@param` / `@return` must add information the signature doesn't already provide. Delete the rest."

Reference: [PHPDoc Tags Reference](https://docs.phpdoc.org/3.0/guide/references/phpdoc/tags/) · [TSDoc](https://tsdoc.org/) · Internal: [`comments-narration`](comments-narration.md)
