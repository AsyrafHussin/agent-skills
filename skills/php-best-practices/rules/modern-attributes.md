---
title: PHP Attributes
impact: CRITICAL
impactDescription: Modern PHP 8.x language features
tags: php, modern-php, php8
---

# PHP Attributes

## Why it matters
Docblock annotations are unparsed strings — typos silently do nothing, IDEs cannot autocomplete them, and static analysis cannot validate their arguments. PHP 8 native attributes are real classes with typed constructors, giving you compile-time safety and full tooling support at zero runtime cost.

## Rule
Use `#[Attribute]` classes for machine-readable metadata. Reserve docblocks for human-readable documentation only.

## Bad
```php
<?php

declare(strict_types=1);

class UserController
{
    /**
     * @Route("/users", methods={"GET"})
     * @Cache(maxage=3600)
     */
    public function index(): void
    {
        // @Route and @Cache are plain strings — a typo or wrong key is invisible
    }
}
```

## Better
```php
<?php

declare(strict_types=1);

#[Attribute]
final class Route
{
    public function __construct(
        public readonly string $path,
        public readonly array $methods = ['GET'],
    ) {}
}

class UserController
{
    #[Route('/users', methods: ['GET'])]
    public function index(): void {}
}
```

## Best
```php
<?php

declare(strict_types=1);

#[Attribute(Attribute::TARGET_METHOD)]
final class Route
{
    public function __construct(
        public readonly string $path,
        public readonly array $methods = ['GET'],
    ) {}
}

#[Attribute(Attribute::TARGET_METHOD)]
final class Cache
{
    public function __construct(
        public readonly int $maxAge = 3600,
    ) {}
}

class UserController
{
    #[Route('/users', methods: ['GET'])]
    #[Cache(maxAge: 600)]
    public function index(): void {}
}

// Reading at runtime via reflection
function resolveRoutes(object $controller): array
{
    $routes = [];

    foreach ((new \ReflectionClass($controller))->getMethods() as $method) {
        foreach ($method->getAttributes(Route::class) as $attr) {
            $route = $attr->newInstance();
            $routes[] = [$route->path, $route->methods, $method->getName()];
        }
    }

    return $routes;
}
```

## Exceptions / trade-offs
Not every metadata concern needs an attribute. If the metadata is only human-readable documentation (e.g., `@param`, `@return`, `@throws`), docblocks are the right tool — they require no reflection overhead and are understood by every IDE and static analyser.

## Attribute target flags

```php
<?php

declare(strict_types=1);

// TARGET restricts where the attribute may be placed
#[Attribute(Attribute::TARGET_CLASS)]
final class Entity
{
    public function __construct(
        public readonly string $table,
    ) {}
}

#[Attribute(Attribute::TARGET_PROPERTY)]
final class Column
{
    public function __construct(
        public readonly string $name,
        public readonly string $type = 'string',
        public readonly bool $nullable = false,
    ) {}
}

#[Attribute(Attribute::TARGET_PROPERTY | Attribute::TARGET_PARAMETER)]
final class Validate
{
    /** @param list<string> $rules */
    public function __construct(
        public readonly array $rules = [],
    ) {}
}

// Using ORM-style attributes on an entity
#[Entity(table: 'users')]
final class User
{
    #[Column(name: 'id', type: 'integer')]
    public int $id;

    #[Column(name: 'email', type: 'string')]
    public string $email;

    #[Column(name: 'created_at', type: 'datetime', nullable: true)]
    public ?DateTimeImmutable $createdAt = null;
}
```

## Reading attributes at runtime

```php
<?php

declare(strict_types=1);

/** @return list<array{path: string, methods: list<string>, handler: array{0: object, 1: string}}> */
function resolveRoutes(object $controller): array
{
    $routes = [];

    foreach ((new \ReflectionClass($controller))->getMethods() as $method) {
        foreach ($method->getAttributes(Route::class) as $attr) {
            $route = $attr->newInstance();
            $routes[] = [
                'path'    => $route->path,
                'methods' => $route->methods,
                'handler' => [$controller, $method->getName()],
            ];
        }
    }

    return $routes;
}
```

## Static-analysis notes
PHPStan and Psalm type-check attribute constructor arguments, validate `TARGET_*` flags, and report unknown attributes. IDEs provide autocompletion and navigation into attribute class definitions.

## Version notes
`PHP 8.0+`

## Related topics
- [modern-named-arguments.md](modern-named-arguments.md) — named arguments make attribute constructors self-documenting
- [modern-override-attribute.md](modern-override-attribute.md) — practical built-in attribute example
