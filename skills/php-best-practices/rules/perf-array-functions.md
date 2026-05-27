---
title: Prefer Native Array Functions, But Profile Before Optimizing
impact: MEDIUM
impactDescription: Runtime performance and memory efficiency
tags: php, performance, optimization
---

# Prefer Native Array Functions, But Profile Before Optimizing

## Why it matters
Manual `foreach` loops that filter, map, or reduce arrays are verbose, harder to compose, and easier to get wrong (off-by-one reindexing, forgotten `break`, mutation of the source array). Native functions express intent clearly and the implementation is in C. That said, most array performance concerns are cargo cult — profile first, rewrite second.

## Rule
Use `array_filter`, `array_map`, `array_reduce`, `array_column`, and `usort` for clarity and idiom. Benchmark only hot paths that process large arrays in tight loops. Readability is the primary reason to prefer them; performance is a secondary benefit.

## Bad
```php
<?php

declare(strict_types=1);

// Manual loops — verbose, harder to compose, mutates result array directly
$active = [];
foreach ($users as $user) {
    if ($user->isActive()) {
        $active[] = $user;
    }
}

$emails = [];
foreach ($users as $user) {
    $emails[] = $user->email;
}

$total = 0.0;
foreach ($orders as $order) {
    $total += $order->total;
}
```

## Better
```php
<?php

declare(strict_types=1);

// Functions used, but without arrow functions — closures are more verbose
$active = array_filter($users, function (User $u): bool {
    return $u->isActive();
});

$emails = array_map(function (User $u): string {
    return $u->email;
}, $users);
```

## Best
```php
<?php

declare(strict_types=1);

// Arrow functions (PHP 7.4+) make intent concise
$active  = array_filter($users, fn(User $u) => $u->isActive());
$emails  = array_map(fn(User $u) => $u->email, $users);
$hasAdmin = (bool) array_filter($users, fn(User $u) => $u->isAdmin());

// Chain filter + map without intermediate named variables
$activeEmails = array_map(
    fn(User $u) => $u->email,
    array_filter($users, fn(User $u) => $u->isActive()),
);

// Reduce for aggregation
$total = array_reduce(
    $orders,
    fn(float $carry, Order $o) => $carry + $o->total,
    0.0,
);

// array_column for key-indexed lookup
$byId = array_column($rows, null, 'id');      // keyed by id, value = full row
$names = array_column($rows, 'name', 'id');   // keyed by id, value = name

// Spread merge (preserves values from right)
$config = [...$defaults, ...$overrides];

// Typed sort with spaceship operator
usort($products, fn(Product $a, Product $b) => $a->price <=> $b->price);
```

## Exceptions / trade-offs
`array_filter` without a callback removes falsy values (including `0` and `''`) — always pass an explicit predicate. `array_map` over large arrays materializes a new array; if memory is a constraint, use a generator instead. For very hot paths on multi-million-item arrays, a plain `foreach` with in-place accumulation can be measurably faster — but measure first.

## Static-analysis notes
PHPStan and Psalm infer return types of `array_filter` and `array_map` when typed arrow functions are used. Without typed callbacks, the inferred return type falls back to `array<int, mixed>`, losing type information.

## Version notes
Arrow functions (`fn() =>`) — PHP 7.4+

## Related topics
- [perf-generators.md](perf-generators.md) — process large arrays one item at a time without materializing them
- [perf-string-functions.md](perf-string-functions.md) — same preference for native functions over manual iteration
