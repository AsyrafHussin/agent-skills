---
title: PHPStan-Style PHPDoc Annotations
impact: CRITICAL
impactDescription: PHPDoc annotations for static analysis
tags: php, phpstan, phpdoc, static-analysis
---

# PHPStan-Style PHPDoc Annotations

## Why it matters

PHP's native type system cannot express generic collections (`list<User>`), fixed-key array structures (`array{id: int, name: string}`), constrained integers (`int<1, 100>`), or conditional return types. Without PHPDoc precision annotations, PHPStan infers `mixed[]` for array parameters and `mixed` for many return types, turning static analysis into educated guesswork. Precision annotations are the difference between PHPStan catching a null dereference at analysis time and that null dereference crashing in production.

## Rule

Use PHPStan-compatible PHPDoc annotations only where native PHP cannot express enough — generic collections, array shapes, class-string constraints, int ranges, and conditional return types. Do not add PHPDoc for types that native declarations already cover. Inline `@var` is a last resort; restructure code to eliminate the ambiguity instead. If a stricter pseudo-type still matches the intended contract, preserve it and fix the proof gap with earlier validation, narrower adapters, or a focused documented ignore instead of weakening the contract.

## Bad

```php
<?php

declare(strict_types=1);

// Untyped arrays — static analysis sees array<mixed>, caller sees nothing
class UserRepository
{
    public function findAll(): array
    {
        return $this->db->fetchAll('SELECT id, name, email FROM users');
    }

    public function paginate(int $page): array
    {
        return [
            'data'  => $this->findAll(),
            'total' => 100,
            'page'  => $page,
        ];
    }

    public function getCount(): int
    {
        return $this->db->count('users');  // could be 0 or more — no constraint expressed
    }
}
```

## Better

```php
<?php

declare(strict_types=1);

// Array shapes on fixed-key structures
class UserRepository
{
    /**
     * @return list<array{id: int, name: string, email: string}>
     */
    public function findAll(): array
    {
        return $this->db->fetchAll('SELECT id, name, email FROM users');
    }

    /**
     * @return array{data: list<array{id: int, name: string, email: string}>, total: int<0, max>, page: int<1, max>}
     */
    public function paginate(int $page): array
    {
        return [
            'data'  => $this->findAll(),
            'total' => $this->db->count('users'),
            'page'  => $page,
        ];
    }

    /** @return int<0, max> */
    public function getCount(): int
    {
        return $this->db->count('users');
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

// Generics with @template for reusable typed collections
/**
 * @template T of object
 */
final class Collection
{
    /** @var list<T> */
    private array $items = [];

    /** @param T $item */
    public function add(object $item): void
    {
        $this->items[] = $item;
    }

    /** @return list<T> */
    public function all(): array
    {
        return $this->items;
    }

    /**
     * @param callable(T): bool $predicate
     * @return static
     */
    public function filter(callable $predicate): static
    {
        $clone        = clone $this;
        $clone->items = array_values(array_filter($this->items, $predicate));
        return $clone;
    }
}

// class-string constrains instantiation to verified class names
/**
 * @template T of HandlerInterface
 * @param class-string<T> $class
 * @return T
 */
function resolveHandler(string $class): HandlerInterface
{
    return new $class();
}

// Conditional return type: return type depends on argument nullability
/**
 * @param string|null $value
 * @return ($value is string ? int : null)
 */
function maybeLength(?string $value): ?int
{
    return $value !== null ? strlen($value) : null;
}

// non-empty-string and non-empty-list signal guaranteed non-emptiness
/**
 * @return non-empty-list<User>
 */
function requireAtLeastOneAdmin(): array
{
    $admins = findAdmins();
    if ($admins === []) {
        throw new \RuntimeException('No admins configured');
    }
    return $admins;
}
```

## String Pseudo-Types

PHP's `string` type is a blunt instrument — it could mean `"admin"`, `""`, `"123"`, or a dangerous SQL statement. PHPStan sharpens this with string pseudo-types that define intent, constraints, and trust boundaries.

| Type | What it guarantees |
|------|--------------------|
| `non-empty-string` | String that is not `''` |
| `numeric-string` | String that represents a number (`"123.45"`) |
| `literal-string` | String known at compile time — no user input |
| `class-string` | A fully-qualified, valid class name |
| `callable-string` | A string that is a callable global function name |

```php
<?php

declare(strict_types=1);

// non-empty-string: guaranteed not ''
/**
 * @param non-empty-string $username
 */
function setUsername(string $username): void
{
    saveToDatabase($username); // PHPStan: safe — cannot be empty
}

setUsername('alice');  // OK
// setUsername('');    // PHPStan error

// literal-string: prevents SQL injection in raw-query wrappers
/**
 * @param literal-string $sql
 */
function runQuery(string $sql): mixed
{
    return $this->db->query($sql); // Only compile-time constants allowed
}

runQuery('SELECT * FROM users');       // OK — literal
// runQuery($_GET['query']);           // PHPStan error — tainted user input

// callable-string: verifies the string resolves to a real callable
/**
 * @param callable-string $callback
 */
function invoke(string $callback): void
{
    $callback(); // PHPStan checks it is actually callable
}

invoke('trim');              // OK
// invoke('undefinedFn');   // PHPStan error

// class-string<T>: constrains to verified class names
/**
 * @template T of Model
 * @param class-string<T> $modelClass
 * @return T
 */
function findFirst(string $modelClass): mixed
{
    return $modelClass::query()->first();
}

$user = findFirst(User::class); // PHPStan infers User return type

// Intersection: literal + domain value
/**
 * @param (literal-string&'database_host') $key
 */
function getConfig(string $key): mixed
{
    return $GLOBALS['config'][$key] ?? null;
}
```

## Numeric and Range Types

PHPStan embeds constraints directly in integer types — `int $page` says nothing; `int<1, 100> $page` says exactly what is valid.

| Type | What it guarantees |
|------|--------------------|
| `positive-int` | Integer > 0 |
| `non-negative-int` | Integer >= 0 |
| `int<min, max>` | Integer within a specific range |
| `int<0, max>` | Non-negative unbounded integer |
| `numeric-string` | String that safely converts to a number |

```php
<?php

declare(strict_types=1);

/** @return positive-int */
function nextId(): int
{
    return $this->sequence->next(); // PHPStan: result is > 0
}

/** @return non-negative-int */
function countItems(): int
{
    return count($this->items); // never negative
}

/** @param int<1, 100> $perPage */
function paginate(int $perPage = 15): Paginator
{
    // $perPage is statically guaranteed to be 1-100
}

/** @return int<0, 255> */
function parseColorChannel(string $hex): int
{
    return hexdec($hex) & 0xFF;
}

/** @param int<1, 65535> $port */
function connectToPort(int $port): void
{
    // valid TCP port range enforced by type
}

/** @param numeric-string $amount */
function parseMoneyString(string $amount): Money
{
    // PHPStan guarantees $amount is a valid numeric string before conversion
    return new Money((int) round(floatval($amount) * 100), Currency::USD);
}
```

## Generics

```php
<?php

declare(strict_types=1);

/**
 * @template TKey of array-key
 * @template TValue
 */
final class TypedMap
{
    /** @var array<TKey, TValue> */
    private array $storage = [];

    /**
     * @param TKey   $key
     * @param TValue $value
     */
    public function set(mixed $key, mixed $value): void
    {
        $this->storage[$key] = $value;
    }

    /**
     * @param TKey $key
     * @return TValue|null
     */
    public function get(mixed $key): mixed
    {
        return $this->storage[$key] ?? null;
    }
}

/**
 * @template T
 * @param list<T>            $items
 * @param callable(T): bool  $predicate
 * @return list<T>
 */
function filterList(array $items, callable $predicate): array
{
    return array_values(array_filter($items, $predicate));
}
```

## Array Shapes

```php
<?php

declare(strict_types=1);

/**
 * @param array{name: string, email: string, age?: int} $data
 * @return array{id: int, name: string, email: string}
 */
function createUser(array $data): array
{
    return [
        'id'    => generateId(),
        'name'  => $data['name'],
        'email' => $data['email'],
    ];
}

/**
 * @param array{
 *     driver: non-empty-string,
 *     host: non-empty-string,
 *     port: int<1, 65535>,
 *     database: non-empty-string,
 *     username: string,
 *     password: string,
 * } $config
 */
function createConnection(array $config): \PDO
{
    return new \PDO(
        "mysql:host={$config['host']};dbname={$config['database']}",
        $config['username'],
        $config['password'],
    );
}
```

## Conditional Return Types

```php
<?php

declare(strict_types=1);

/**
 * @param string|null $value
 * @return ($value is string ? int : null)
 */
function maybeLength(?string $value): ?int
{
    return $value !== null ? strlen($value) : null;
}

/**
 * @template T
 * @param T|null $value
 * @param T      $default
 * @return T
 */
function coalesce(mixed $value, mixed $default): mixed
{
    return $value ?? $default;
}
```

## Exceptions / trade-offs

- **Do not annotate what native types already express.** `public function getId(): int` needs no `@return int` PHPDoc — the native declaration is the contract.
- **Inline `@var` is a last resort.** If you need `/** @var User $user */` before a variable, the better fix is to restructure the code so PHPStan can infer the type from context.
- **`@param array $data`** with no shape annotation is worse than nothing — it silently widens the type to `mixed[]`. Either add a shape or use a typed DTO.
- PHPDoc on internal private helpers with obvious types adds noise without value.

## Static-analysis notes

- **`list<T>`** is a PHPStan/Psalm type for sequential arrays (keys 0, 1, 2…). Use it instead of `array<int, T>` for ordered collections.
- **`array{key: Type, ...}`** — PHPStan verifies key existence and types at every access site.
- **`class-string<T>`** — PHPStan verifies the string is a valid, instantiable class name that is a subtype of `T`.
- **`int<min, max>`** — PHPStan propagates the range constraint through arithmetic and comparisons.
- **`@template`** gives full generic safety at zero runtime cost. PHPStan infers the concrete type at each call site.
- **`non-empty-string`** and **`non-empty-list<T>`** eliminate the need for defensive `=== ''` or `=== []` guards after the annotation point.
- **`literal-string`** is the PHPStan mechanism for preventing user-supplied strings from reaching SQL/shell/eval functions.
- **Do not weaken honest strict contracts to appease the analyzer.** Relax `literal-string`, `class-string<T>`, `non-empty-string`, and similar pseudo-types only when the contract itself is wrong, not merely because the current code path does not prove it yet.
- **`positive-int`** and **`non-negative-int`** are shorthand for `int<1, max>` and `int<0, max>` respectively.
- PHPStan level 7+ requires `@param`/`@return` annotations when native types are absent; level 9 flags all `mixed` usage.

## Related topics

- [type-mixed-avoid.md](type-mixed-avoid.md) — use precise annotations instead of leaving `mixed` in place
- [design-value-objects.md](design-value-objects.md) — value objects often eliminate the need for complex array shapes
- [tooling-phpstan-cs-fixer.md](tooling-phpstan-cs-fixer.md) — run PHPStan to validate annotations in CI
- [type-union-types.md](type-union-types.md) — prefer native union types over PHPDoc unions where PHP supports it
- [sec-sql-prepared.md](sec-sql-prepared.md) — `literal-string` complements prepared statements for SQL safety
