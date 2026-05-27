---
title: First-Class Callable Syntax
impact: CRITICAL
impactDescription: Modern PHP 8.x language features
tags: php, modern-php, php8
---

# First-Class Callable Syntax

## Why it matters
Array callables like `[$obj, 'method']` are opaque strings at the type level — renaming the method in an IDE does not update them, static analysis cannot resolve their signature, and a typo produces a runtime error. First-class callables are real closures with fully-resolved types available at the call site.

## Rule
Use the `...` first-class callable syntax instead of array callables or `Closure::fromCallable()`. The result is a typed `Closure` that IDEs and static analysers can inspect.

## Bad
```php
<?php

declare(strict_types=1);

final class Formatter
{
    public function format(string $value): string
    {
        return strtoupper(trim($value));
    }
}

$formatter = new Formatter();

// String-based — not refactorable, no type info, runtime error on typo
$callable = [$formatter, 'format'];

$results = array_map($callable, $names);
```

## Better
```php
<?php

declare(strict_types=1);

final class Formatter
{
    public function format(string $value): string
    {
        return strtoupper(trim($value));
    }
}

$formatter = new Formatter();

// Explicit, but verbose
$callable = Closure::fromCallable([$formatter, 'format']);

$results = array_map($callable, $names);
```

## Best
```php
<?php

declare(strict_types=1);

final class Formatter
{
    public function format(string $value): string
    {
        return strtoupper(trim($value));
    }
}

$formatter = new Formatter();

// Concise, refactorable, fully typed Closure
$results = array_map($formatter->format(...), $names);

// Works for static methods and plain functions too
$trimmed = array_map(trim(...), $names);
```

## Exceptions / trade-offs
- When the method name is determined at runtime (dynamic dispatch), the array callable `[$obj, $methodName]` remains necessary. Document why and validate the name.

## Pipeline and composition

```php
<?php

declare(strict_types=1);

// Pipeline pattern using first-class callables
final class Pipeline
{
    /** @var list<\Closure> */
    private array $stages = [];

    public function pipe(\Closure $stage): static
    {
        $clone = clone $this;
        $clone->stages[] = $stage;
        return $clone;
    }

    public function process(mixed $payload): mixed
    {
        return array_reduce(
            $this->stages,
            fn(mixed $carry, \Closure $stage): mixed => $stage($carry),
            $payload,
        );
    }
}

final class StringProcessor
{
    public function normalise(string $value): string
    {
        return strtolower(trim($value));
    }

    public function slugify(string $value): string
    {
        return str_replace(' ', '-', $this->normalise($value));
    }
}

$processor = new StringProcessor();

$pipeline = (new Pipeline())
    ->pipe(trim(...))
    ->pipe(strtolower(...))
    ->pipe($processor->normalise(...));

$result = $pipeline->process('  Hello World  '); // 'hello world'
```

## Event listeners and higher-order functions

```php
<?php

declare(strict_types=1);

final class UserController
{
    public function __construct(
        private readonly EventDispatcher $dispatcher,
        private readonly UserService $service,
    ) {
        // Subscribe instance methods as typed closures
        $this->dispatcher->subscribe('user.created', $this->onUserCreated(...));
        $this->dispatcher->subscribe('user.deleted', $this->onUserDeleted(...));
    }

    private function onUserCreated(User $user): void {}
    private function onUserDeleted(UserId $id): void {}
}

// Sorting with first-class callables
/** @var list<User> $users */
$sorted = array_filter($users, $someUser->isActive(...));

usort($users, fn(User $a, User $b) => $a->name <=> $b->name);
```

## Built-in PHP functions as callables

```php
<?php

declare(strict_types=1);

$strings = ['  hello  ', ' world ', '  php  '];

// Native functions as first-class callables
$trimmed   = array_map(trim(...),      $strings);
$lengths   = array_map(strlen(...),    $trimmed);
$uppercased = array_map(strtoupper(...), $trimmed);

// Static methods
$formatDate = \DateTimeImmutable::createFromFormat(...);
$dates = array_map(
    fn(string $s) => \DateTimeImmutable::createFromFormat('Y-m-d', $s),
    ['2024-01-01', '2024-06-15'],
);
```

## Static-analysis notes
PHPStan and Psalm resolve the exact `Closure` type (including parameter and return types) from first-class callables. This enables accurate type inference for `array_map`, `array_filter`, and similar higher-order functions. Psalm's `@param callable(T): R` annotations work fully with first-class callable results.

## Version notes
`PHP 8.1+`

## Related topics
- [modern-arrow-functions.md](modern-arrow-functions.md) — lightweight closures for simple transforms
- [modern-pipe-operator.md](modern-pipe-operator.md) — composing callables in a pipeline (PHP 8.5+)
