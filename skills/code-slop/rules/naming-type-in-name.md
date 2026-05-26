---
title: Type Encoded in Variable Names
impact: MEDIUM
impactDescription: "Embeds type in the identifier when the type system already does it; ages badly with refactors"
tags: naming, hungarian, type-in-name, ai-fingerprint
---

## Type Encoded in Variable Names

**Impact: MEDIUM (Embeds type in the identifier when the type system already does it; ages badly with refactors)**

`userObject`, `resultArray`, `stringData`, `listOfItems`, `userIdString`, `dataMap` — these names encode the *type* of the value, redundantly with the actual type declaration. Both PHP 8+ and TypeScript have rich type systems that show the type in the signature. The name should encode what the value **means**, not what its shape is.

The pattern is a tell because it suggests the author optimised for "obvious from any one line" rather than "obvious in context". A human author trusts the reader to look at the signature.

## Incorrect

```php
// ❌ Type-in-name everywhere

public function processItemsList(array $itemsArray, int $userIdInteger): array
{
    $resultArray = [];
    foreach ($itemsArray as $itemObject) {
        $priceFloat = $itemObject->priceFloat;
        $resultArray[] = $priceFloat;
    }
    return $resultArray;
}
```

```typescript
// ❌ Same in TS
function getUserDataObject(userIdString: string): UserDataResponseObject {
  const responseObject = await api.fetch(userIdString);
  const dataArray = responseObject.dataArray;
  const userObjectList = dataArray.map((itemObject: UserObject) => itemObject);
  return userObjectList;
}
```

**Why it's slop:**
- The type is declared on the parameter and visible in the signature
- Renaming `$itemsArray` to a `Collection` requires updating every variable reference
- The name `$itemObject` is redundant — the static type says it's a `User`
- Pattern is a clear "tutorial-blog" signature; teammates don't write like this

## Correct

```php
// ✅ Names encode meaning, not shape

public function totalsFor(array $items, int $userId): array
{
    return array_map(fn (Item $item) => $item->price, $items);
}
```

```typescript
// ✅ Domain words, types in the signature
async function fetchUser(userId: string): Promise<User> {
  const { data } = await api.fetch(userId);
  return data;
}
```

**Why it reads human:**
- Each name describes the role (`items`, `userId`, `total`) not the shape
- Type info is in the type system, where refactors update it automatically
- If `items` becomes a `Collection<Item>`, the variable name still reads correctly

## When type-in-name IS okay

Conventions where the type is part of the meaning:

- **`*Id`** suffix — `userId` (distinguishes from `user` which is the whole object) ✓
- **`*Count`** suffix — `userCount` ✓
- **`is*`** / **`has*`** / **`can*`** booleans — `isPaid`, `hasShipping`, `canEdit` ✓
- **Maps with two-type names** — `usersByEmail` (carries both types in one word) ✓
- **DTO classes** — `UserResponse`, `OrderPayload` (clarifies "this is the wire form, not the domain model") ✓

The suspect patterns are `*Object`, `*Array`, `*List`, `*Map`, `*String`, `*Integer`, `*Float` on local variables.

## Detection

```bash
# Suspect type-suffix names on variables (PHP)
grep -rEn '\$[a-z][a-zA-Z]*?(Object|Array|List|Map|String|Integer|Float|Boolean|Bool)\b' \
  --include='*.php' app/

# TS / JS
grep -rEn '\b(const|let)\s+[a-z][a-zA-Z]*?(Object|Array|List|Map|String|Boolean)\b' \
  --include='*.ts' --include='*.tsx' --include='*.js' --include='*.jsx' src/
```

A few hits are fine (especially in test data setups). **Density across multiple files** is the signal — a project with 30+ `*Array` / `*Object` variables almost certainly came from a model.

Reference: [Clean Code — Chapter 2](https://www.oreilly.com/library/view/clean-code/9780136083238/) · [Joel Spolsky — Making Wrong Code Look Wrong](https://www.joelonsoftware.com/2005/05/11/making-wrong-code-look-wrong/) (the "useful Hungarian" original — different from this rule's anti-pattern) · Internal: [`naming-over-descriptive`](naming-over-descriptive.md)
