---
title: Narration Comments
impact: CRITICAL
impactDescription: "The single loudest AI tell; cuts reading speed and signals the author didn't read what they shipped"
tags: comments, narration, ai-fingerprint
---

## Narration Comments

**Impact: CRITICAL (The single loudest AI tell; cuts reading speed and signals the author didn't read what they shipped)**

Models love narrating the next line in a comment above it. The comment adds no information — it just restates the code in English. A reader has to read both, decide they say the same thing, and feel a tiny tax of distrust. Across a 500-line file, the tax compounds into "I have no idea what this codebase is doing."

A useful comment answers **why**, not **what**. If the code itself says *what*, the comment is noise.

## Incorrect

```php
// ❌ Comments narrate the next line — pure AI fingerprint

public function createUser(array $data): User
{
    // Validate the input data
    $validated = Validator::make($data, [...])->validate();

    // Create a new user with the validated data
    $user = User::create($validated);

    // Send a welcome email to the user
    Mail::to($user)->send(new WelcomeEmail($user));

    // Return the newly created user
    return $user;
}
```

```typescript
// ❌ Same pattern in TS — every line narrated above
function calculateTotal(items: Item[]): number {
  // Initialize the total to zero
  let total = 0;

  // Loop through each item in the items array
  for (const item of items) {
    // Add the item's price multiplied by quantity to the total
    total += item.price * item.quantity;
  }

  // Return the calculated total
  return total;
}
```

**Why it's slop:**
- Every comment is a literal English translation of the code on the next line
- A reader's eyes have to traverse twice (comment → code) to learn nothing extra
- Suggests the author was thinking out loud rather than thinking ahead
- A human author who knew this code wouldn't bother writing these comments

## Correct

```php
// ✅ Comments removed (the code says what it does); explain WHY only if non-obvious

public function createUser(array $data): User
{
    $validated = Validator::make($data, [...])->validate();

    $user = User::create($validated);
    Mail::to($user)->send(new WelcomeEmail($user));

    return $user;
}
```

```typescript
// ✅ Total comments removed; if any line needed comment, it'd be a WHY
function calculateTotal(items: Item[]): number {
  return items.reduce((sum, i) => sum + i.price * i.quantity, 0);
}
```

If a comment IS needed, it explains *why*:

```typescript
// ✅ Good comment — explains a hidden constraint
// Note: `total` is in cents (Stripe API uses integer minor units).
function calculateTotal(items: Item[]): number {
  // Stripe rejects single charges > 999,999_99 cents (= $999,999.99). Cap to avoid charge failure.
  const total = items.reduce((sum, i) => sum + i.price * i.quantity, 0);
  return Math.min(total, 99_999_999);   // 999,999.99 expressed in cents
}
```

**Why it reads human:**
- Code speaks for itself; reader processes once
- The one remaining comment carries a *load-bearing* piece of context (Stripe constraint) you couldn't infer from the code
- Signals the author understood the code well enough to know what didn't need explaining

## Detection

```bash
# Heuristic: comments immediately followed by code that uses the comment's keyword
# (rough but effective — flags lines where the comment is one word-of-code below)
grep -rEn '^[[:space:]]*//.*' --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' src/ | head -50
grep -rEn '^[[:space:]]*//.*' --include='*.php' app/ | head -50

# Stronger heuristic: comment + next non-blank line share a keyword
# (run in code review tools or PHPStorm "Comment Density" inspections)

# Easiest enforcement: ESLint capital-comments / no-inline-comments rules combined
# with a team-level "comments answer WHY only" code-review checklist.
```

There is no fully automatic linter for this — it requires a reviewer's judgement. The detection pattern is "does the comment translate, or add context?"

Reference: [Code Complete — Self-Documenting Code](https://www.microsoftpressstore.com/store/code-complete-9780735619678) · Internal: [`comments-empty-docblocks`](comments-empty-docblocks.md)
