---
title: Null Checks for Impossible Nulls
impact: HIGH
impactDescription: "Defensive code in places that can't fail; clutters the path and signals model didn't trust the type system"
tags: defensive-programming, null-checks, type-safety, ai-fingerprint
---

## Null Checks for Impossible Nulls

**Impact: HIGH (Defensive code in places that can't fail; clutters the path and signals model didn't trust the type system)**

When the type system or surrounding code already guarantees a value is non-null, a defensive null check adds noise and tells the reader "the author wasn't sure". Common cases:

- A non-nullable parameter (typed `User $user`, not `?User $user`) — can't be null
- After `Model::findOrFail()` — never returns null (throws on miss)
- After a non-null assertion in the previous line
- Inside a `forEach` / `map` loop where the iterable can't contain null

The pattern signals the model defaulted to "check everything for safety" without reading the types around it.

## Incorrect

```php
// ❌ Null check after findOrFail — findOrFail throws if not found

public function show(int $id): View
{
    $user = User::findOrFail($id);

    if ($user === null) {
        abort(404);
    }

    return view('users.show', compact('user'));
}

// ❌ Null check on a non-nullable parameter

public function notify(User $user, string $message): void  // $user is required
{
    if ($user === null) {
        return;
    }
    Mail::to($user)->send(new GenericNotification($message));
}

// ❌ Null check repeated in same function
public function process(Order $order): void
{
    if ($order === null) return;
    $items = $order->items;
    if ($order === null) return;     // already checked above; flow can't make it null
    foreach ($items as $item) {
        if ($order === null) return; // still impossible
        // ...
    }
}
```

```typescript
// ❌ TS: defensive checks the compiler already enforced

function greet(user: User): string {       // `user: User` — non-null in type system
  if (!user) return '';                    // dead branch; TS would error
  return `Hi, ${user.name}`;
}

// ❌ Defensive after a guard
function process(user: User | null): void {
  if (!user) return;                       // narrows to User
  if (user) {                              // always true after the guard
    doStuff(user);
  }
}

// ❌ Optional chaining on a definitely-defined value
const userId = currentUser.id;             // currentUser is non-nullable here
console.log(currentUser?.email);           // ?. is unnecessary
```

**Why it's slop:**
- The dead branch is dead — never executes; reviewers must mentally rule it out
- Signals the author didn't trust the type system
- `findOrFail` is well-known Laravel API; checking its result for null says "I read the docs but don't believe them"
- Stacking redundant guards in one function (the third example) is unmistakable

## Correct

```php
// ✅ Trust findOrFail — it throws, doesn't return null

public function show(int $id): View
{
    return view('users.show', [
        'user' => User::findOrFail($id),
    ]);
}

// ✅ Non-nullable parameter — no check needed
public function notify(User $user, string $message): void
{
    Mail::to($user)->send(new GenericNotification($message));
}
```

```typescript
// ✅ Trust the types
function greet(user: User): string {
  return `Hi, ${user.name}`;
}

// ✅ Single guard; TS narrows the type below
function process(user: User | null): void {
  if (!user) return;
  doStuff(user);   // TS knows user is User here
}

// ✅ No optional chaining when not optional
console.log(currentUser.email);
```

**Why it reads human:**
- Each line carries weight; no dead branches
- Type system does the work, not runtime checks
- Reader can scan past the function without ruling out impossible paths

## When null checks ARE warranted

- **At trust boundaries** — input from HTTP/queue/file, before the type narrows
- **On nullable returns** from third-party libraries (e.g., `Eloquent::find()` returns `?Model`)
- **In `try {} catch` flows** where the value might be partially constructed
- **When the type really IS nullable** — `?User $user` parameters that the caller can leave empty

The test: **does the type say it could be null?** If yes, check. If no, trust.

## Detection

```bash
# Null checks immediately after findOrFail in PHP — a recognisable AI signature
# (Two-pass: find files with findOrFail, then look for null checks inside them.)
grep -rl 'findOrFail' --include='*.php' app/ | xargs grep -EnB1 'if\s*\(\s*\$[a-zA-Z_]+\s*===?\s*null\s*\)' 2>/dev/null

# TS: optional chaining on values typed as non-null
# (best caught via TS strict mode: noUncheckedIndexedAccess + strictNullChecks)
npx tsc --noEmit --strict

# PHPStan level 8+ catches many "always-false condition" cases
vendor/bin/phpstan analyse --level=9
```

**PHPStan level 9+ (level 10 is max in PHPStan 2.0) and TypeScript strict mode are your best mechanical defenses** — both will flag "condition is always false" or "value is never null". Adopt them and most of these impossible-null checks disappear from new code automatically.

Reference: [PHPStan rule levels](https://phpstan.org/user-guide/rule-levels) · [TS strict mode](https://www.typescriptlang.org/tsconfig#strict) · Internal: [`defensive-generic-catch`](defensive-generic-catch.md), [`defensive-missing-real`](defensive-missing-real.md)
