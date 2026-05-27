---
title: Arrow Functions
impact: CRITICAL
impactDescription: Modern PHP 8.x language features
tags: php, modern-php, php8
---

# Arrow Functions

## Why it matters
Verbose closures with `use` capture lists add noise that buries the actual transform. Arrow functions eliminate the `use` clause by capturing the outer scope implicitly, making simple expression-level transforms read as clearly as the math they represent. The danger is the opposite failure: packing multi-statement logic into an arrow function to avoid naming it, producing unreadable one-liners.

## Rule
Use arrow functions only for tiny, single-expression transforms with no side effects. Anything with branching, multiple statements, or side effects belongs in a named method or a regular closure.

## Bad
```php
<?php

declare(strict_types=1);

$multiplier = 3;

// Verbose closure — $multiplier capture is boilerplate noise
$result = array_map(
    function (int $n) use ($multiplier): int {
        return $n * $multiplier;
    },
    $numbers,
);
```

## Better
```php
<?php

declare(strict_types=1);

$multiplier = 3;

// Arrow function: outer scope captured implicitly, expression is clear
$result = array_map(
    fn (int $n): int => $n * $multiplier,
    $numbers,
);
```

## Best
```php
<?php

declare(strict_types=1);

// GOOD: arrow function for a clean, focused transform
$multiplier = 3;
$result = array_map(fn (int $n): int => $n * $multiplier, $numbers);

// BAD: do NOT do this — complex logic forced into an arrow function
$processed = array_map(
    fn (Order $o): string => $o->isPaid()
        ? ($o->isShipped() ? 'delivered' : 'processing')
        : ($o->isCancelled() ? 'cancelled' : 'pending'),
    $orders,
);

// BETTER: extract to a named method when logic is non-trivial
$processed = array_map(fn (Order $o): string => $o->statusLabel(), $orders);
```

## Exceptions / trade-offs
- Any logic with side effects, I/O, or more than one expression — use a named method or a regular closure instead.
- Arrow functions cannot contain `return` statements; if you need early returns or guards, use a closure or a method.
- Multi-step pipelines may benefit from named methods over chained arrow functions to keep stack traces readable.

## Practical array operation patterns

```php
<?php

declare(strict_types=1);

$multiplier = 3;
$numbers    = [1, 2, 3, 4, 5];

// Map, filter, sort with captured outer scope
$tripled = array_map(fn (int $n): int => $n * $multiplier, $numbers);
$evens   = array_filter($numbers, fn (int $n): bool => $n % 2 === 0);

// Sorting
/** @var list<User> $users */
usort($users, fn (User $a, User $b): int => $a->name <=> $b->name);

/** @var list<Product> $products */
usort($products, fn (Product $a, Product $b): int => $a->price() <=> $b->price());

// Collection extraction
$activeEmails = array_map(
    fn (User $u): string => $u->email(),
    array_filter($users, fn (User $u): bool => $u->isActive()),
);

// Higher-order factory: return an arrow function from a function
function multiplierOf(int $factor): \Closure
{
    return fn (int $n): int => $n * $factor;
}

$double = multiplierOf(2);
$triple = multiplierOf(3);

array_map($double, $numbers); // [2, 4, 6, 8, 10]
array_map($triple, $numbers); // [3, 6, 9, 12, 15]
```

## Static-analysis notes
PHPStan and Psalm infer the return type from the arrow function body expression. Explicit return type annotations (`: int`, `: string`) are recommended for public APIs and non-obvious transforms; they are optional for trivially obvious cases.

## Version notes
`PHP 7.4+`

## Related topics
- [modern-first-class-callables.md](modern-first-class-callables.md) — reference existing methods as callables without a wrapping arrow function
- [modern-pipe-operator.md](modern-pipe-operator.md) — composing transforms in sequence
