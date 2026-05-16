---
title: Generic Placeholder Names
impact: CRITICAL
impactDescription: "Reduces every variable to 'a thing' — kills the most powerful form of self-documentation"
tags: naming, generic-names, ai-fingerprint
---

## Generic Placeholder Names

**Impact: CRITICAL (Reduces every variable to 'a thing' — kills the most powerful form of self-documentation)**

`data`, `result`, `info`, `temp`, `value`, `item`, `helper`, `manager`, `utils` — these are the names a model picks when it doesn't know what the value represents in the domain. They pass linters, type checks, every metric. They cost the reader real attention to track ("which `data` is this?").

Names are the cheapest, highest-leverage form of self-documentation in any codebase. A model that names every intermediate variable `data` or `result` is signalling it didn't think about the domain.

## Incorrect

```php
// ❌ Every variable is generic

public function exportUsers(): Collection
{
    $data = User::all();

    $result = [];
    foreach ($data as $item) {
        $info = [
            'name' => $item->name,
            'email' => $item->email,
        ];
        $result[] = $info;
    }

    return collect($result);
}
```

```typescript
// ❌ Same pattern
async function fetchOrders(userId: string) {
  const data = await api.getOrders(userId);
  const result = data.map(item => {
    const info = {
      id: item.id,
      total: item.total,
    };
    return info;
  });
  return result;
}
```

**Why it's slop:**
- Every variable is "a thing"; reader has no anchor to the domain
- `$item` inside a loop over `$data` is doubly opaque — what kind of item? what kind of data?
- A reader hitting line 50 of this file can't tell what `$result` holds without tracing the whole function
- A human author who lived with this code would use domain words

## Correct

```php
// ✅ Names tell you what the values are in the domain

public function exportUsers(): Collection
{
    return User::all()
        ->map(fn (User $user) => [
            'name'  => $user->name,
            'email' => $user->email,
        ]);
}
```

```typescript
// ✅ Domain words everywhere
async function fetchOrders(userId: string): Promise<OrderSummary[]> {
  const orders = await api.getOrders(userId);
  return orders.map(order => ({
    id:    order.id,
    total: order.total,
  }));
}
```

**Why it reads human:**
- A reader on any line knows the domain object in scope
- Domain names compose — `orders.map(order => …)` reads as a sentence
- The original `$data → $item → $info → $result` chain collapses into one expression because the names made the intermediate variables unnecessary

## When generic names ARE OK

A handful of names are conventional, short-scope, and fine:

- **`i`, `j`, `k`** — loop indices in a 3-line `for`
- **`x`, `y`, `z`** — math/geometry (coordinates)
- **`_`** — explicit "ignored value"
- **`acc`** — accumulator in a `reduce` callback
- **`prev`, `next`** — in middleware / chained handlers where they're language conventions
- **`req`, `res`** — Express handlers (don't fight a framework convention)

If the value flows through 5+ lines or escapes the immediate function, use a domain name.

## Detection

```bash
# Bare local-variable declarations using generic names (PHP)
grep -rEn '\$(data|result|info|temp|item|helper|value)\b' --include='*.php' app/ | wc -l

# TS/JS — generic const/let
grep -rEn '\b(const|let)\s+(data|result|info|temp|helper|value|item)\b' --include='*.ts' --include='*.tsx' --include='*.js' src/ | wc -l

# Density signal: files with > 10 generic-name hits
# (in a 200-line file, 10+ generic names is suspicious)
```

There's no fully automatic detector — judgment is required. The signal is **density**: a few are fine, a thicket is a slop fingerprint.

Reference: [Clean Code (Robert C. Martin) — Chapter 2: Meaningful Names](https://www.oreilly.com/library/view/clean-code/9780136083238/) · Internal: [`naming-over-descriptive`](naming-over-descriptive.md)
