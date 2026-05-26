---
title: Trivial Boilerplate
impact: MEDIUM
impactDescription: "Pattern-matched boilerplate that hides intent and inflates line count"
tags: style, boilerplate, ai-fingerprint, readability
---

## Trivial Boilerplate

**Impact: MEDIUM (Pattern-matched boilerplate that hides intent and inflates line count)**

The class of "code that says less than the underlying expression". Common AI variants:

- `if (x) return true; else return false;` — when `return x` does it
- `return x === true ? true : false;` — ditto
- `const isPaid: boolean = order.status === 'paid';` — TS infers `boolean` already; the annotation is noise
- `const name: string = 'asyraf';` — TS infers `string`
- `await Promise.resolve(value)` — when `value` is already non-Promise
- `try { await fn(); } catch (e) { throw e; }` — pass-through catch
- `[...array]` to "make a copy" when the next operation doesn't mutate

Each is a small AI fingerprint. A few are fine. A repo with many of them across files reads as model-generated.

## Incorrect

```typescript
// ❌ if-true-false
function isPaid(order: Order): boolean {
  if (order.status === 'paid') {
    return true;
  } else {
    return false;
  }
}

// ❌ Ternary returning the booleans it was given
function isCompleted(order: Order): boolean {
  return order.status === 'completed' ? true : false;
}

// ❌ Redundant type annotations on obvious literals
const name: string = 'asyraf';
const age: number = 30;
const isActive: boolean = true;
const orders: Order[] = await db.orders.find();   // db.orders.find() return type IS Order[]

// ❌ Pass-through catch
async function processPayment(order: Order): Promise<Charge> {
  try {
    return await stripe.charges.create({ /* ... */ });
  } catch (e) {
    throw e;                                       // catch literally does nothing
  }
}

// ❌ Unnecessary await + Promise.resolve
async function formatName(user: User): Promise<string> {
  return await Promise.resolve(`${user.first} ${user.last}`);
}

// ❌ Spread to "copy" — but nothing mutates
function totalsFor(items: Item[]): number {
  const copied = [...items];                       // unused: the next op doesn't mutate
  return copied.reduce((s, i) => s + i.price, 0);
}
```

```php
// ❌ PHP equivalents
public function isPaid(Order $order): bool
{
    if ($order->status === 'paid') {
        return true;
    } else {
        return false;
    }
}

public function processWebhook(Request $request): JsonResponse
{
    try {
        return $this->handle($request);
    } catch (\Exception $e) {
        throw $e;                                  // pass-through
    }
}
```

**Why it's slop:**
- Each line carries no information beyond the simpler form
- The `try { ... } catch (e) { throw e; }` is the most embarrassing — six tokens to do nothing
- Type annotations on inferred literals fight the type system instead of using it
- A reader's eyes have to traverse "what does this expand to?" rather than read the expression directly

## Correct

```typescript
// ✅ Direct expressions
function isPaid(order: Order): boolean {
  return order.status === 'paid';
}

function isCompleted(order: Order): boolean {
  return order.status === 'completed';
}

// ✅ Let TS infer
const name = 'asyraf';
const age = 30;
const isActive = true;
const orders = await db.orders.find();

// ✅ No catch when not handling
async function processPayment(order: Order): Promise<Charge> {
  return stripe.charges.create({ /* ... */ });
}

// ✅ No need to await a non-Promise
function formatName(user: User): string {
  return `${user.first} ${user.last}`;
}

// ✅ No spread when nothing mutates
function totalsFor(items: Item[]): number {
  return items.reduce((s, i) => s + i.price, 0);
}
```

```php
// ✅ PHP equivalents
public function isPaid(Order $order): bool
{
    return $order->status === 'paid';
}

public function processWebhook(Request $request): JsonResponse
{
    return $this->handle($request);
}
```

**Why it reads human:**
- Each expression does exactly the work; nothing extra
- Type inference does its job; the type annotations are added where they earn their place (function signatures, public APIs)
- No pass-through catch; if an error needs handling, it's handled

## When annotations / spreads ARE worth it

A few cases:

- **Public-API parameters / returns** — explicit types document the contract: `function add(a: number, b: number): number`
- **Complex inferred types** — annotate when the inferred type is hard to read
- **Boundary code** — `as const`, `satisfies`, or explicit annotations on configuration objects help downstream type narrowing
- **Spread when you actually want a copy** — before sorting, before mutating, before passing to a function that mutates

The trivial-boilerplate test: **does removing the construct change the program's behaviour or the reader's understanding?** If no, remove.

## Detection

```bash
# if-true-false
grep -rEnB1 -A2 'if\s*\([^)]+\)\s*\{?\s*$' --include='*.ts' --include='*.tsx' --include='*.js' --include='*.php' \
  src/ app/ 2>/dev/null | grep -B2 -A1 'return true\b' | grep -A2 'else' | head

# Pass-through catch
grep -rEnB0 -A1 'catch\s*\([^)]*\)\s*\{' --include='*.ts' --include='*.tsx' --include='*.php' \
  src/ app/ 2>/dev/null | grep -B1 '^\s*throw\b'

# Ternary returning booleans
grep -rEn '\?\s*true\s*:\s*false\b' --include='*.ts' --include='*.tsx' --include='*.js' --include='*.php' \
  src/ app/

# Redundant type annotations on string/number/boolean literals (TS)
grep -rEn ':\s*(string|number|boolean)\s*=\s*("[^"]*"|[0-9]+|true|false)' --include='*.ts' --include='*.tsx' src/
```

ESLint rules:

```json
{
  "rules": {
    "no-useless-return": "error",
    "no-useless-catch": "error",
    "no-unneeded-ternary": "error"
  }
}
```

Reference: [ESLint — no-useless-catch / ternary / return](https://eslint.org/docs/latest/rules/) · [PHP-CS-Fixer rules](https://cs.symfony.com/) · Internal: [`defensive-generic-catch`](defensive-generic-catch.md)
