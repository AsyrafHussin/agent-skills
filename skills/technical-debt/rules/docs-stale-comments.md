---
title: Stale Comments
impact: MEDIUM
impactDescription: "Wrong information is worse than no information"
tags: documentation, comments, maintenance
---

## Stale Comments

**Impact: MEDIUM (Wrong information is worse than no information)**

A comment that contradicts the code it describes actively misleads readers. They either trust the comment and write a bug, or they distrust all comments and miss the load-bearing ones. Stale comments are negative-value documentation.

## How to Detect

There is no perfect tool — stale comments are read-and-judge. Useful starting points:

```bash
# Find comments referencing functions, classes, files that no longer exist
grep -rEn '@see |@deprecated |TODO\\(|see also' src/ | \
  while IFS= read -r line; do
    REF=$(echo "$line" | grep -oE '[A-Z][a-zA-Z]+::[a-zA-Z]+|[a-z_]+\\.[a-z]+')
    [ -n "$REF" ] && ! grep -rq "$REF" src/ && echo "STALE: $line"
  done

# Find comments referencing removed parameters
# (compare comment params with current function signature)
```

Hotspot: comments next to code that has been git-touched more recently than the comment.

## Incorrect

```typescript
// ❌ Comment lies about behaviour
/**
 * Returns the user's full name in "Last, First" format.
 */
function displayName(user: User): string {
  return `${user.firstName} ${user.lastName}`;   // actually "First Last"
}

// ❌ Comment references removed parameter
/**
 * @param userId - the user to load
 * @param includeArchived - whether to include archived records
 */
function loadUser(userId: string) {              // includeArchived removed last year
  return db.users.findOne({ id: userId, archived: false });
}

// ❌ "Temporary" workaround that became permanent
// HACK: workaround for Stripe API bug — remove after their 2022 fix
const tax = Math.round(subtotal * 0.06 * 100) / 100;
```

**Problems:**
- A reader follows the comment, writes integration code expecting "Last, First", ships a bug
- IDE autocomplete picks up the stale `@param`, suggesting a parameter that no longer exists
- "Temporary" workaround now load-bearing; nobody dares remove it

## Correct

```typescript
// ✅ Comment matches reality, or is deleted
/**
 * Returns "First Last" — used in user-facing greetings.
 */
function displayName(user: User): string {
  return `${user.firstName} ${user.lastName}`;
}

// ✅ Parameter doc removed when parameter removed
function loadUser(userId: string) {
  return db.users.findOne({ id: userId, archived: false });
}

// ✅ Either remove the workaround, or update the comment with current rationale
// Stripe's 2022 fix shipped; this rounding is kept because our DB stores 4 decimal places
// and accounting reports require 2-decimal-place reconciliation. (#TAX-431)
const tax = Math.round(subtotal * 0.06 * 100) / 100;
```

**Benefits:**
- Comments become trusted again
- IDE hints align with reality
- "Why does this exist" is captured at the right level of fidelity

## Remediation Strategy

- **Effort:** S per comment
- **When to pay down:** On every PR — if you touch code, read the surrounding comments and verify or delete. Reviewers should call out stale comments next to changed lines.
- **Heuristic:** when in doubt, delete. Code documents *what*; the commit message documents *why*. A stale comment is rarely the right tool to keep.

Reference: [John Ousterhout — A Philosophy of Software Design, Ch. 13](https://web.stanford.edu/~ouster/cgi-bin/aposd.php)
