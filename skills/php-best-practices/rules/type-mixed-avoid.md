---
title: Avoid Mixed Type
impact: CRITICAL
impactDescription: Strong type safety across all PHP code
tags: php, types, strict-types, type-safety
---

# Avoid Mixed Type

## Why it matters
`mixed` is a promise to the type system that you will handle anything — and a promise the type system cannot help you keep. Every `mixed` in a signature forces every downstream caller to guard against every possible type before they can do useful work. Over time `mixed` accumulates: one becomes five, five become ten, and the codebase loses static-analysis coverage precisely in the paths most likely to carry bugs.

## Rule
Treat `mixed` as a temporary border checkpoint, not a home address. Use it only at true external boundaries (JSON decode output, framework callbacks, deserialization), and narrow it to a specific type immediately after entry. Never propagate `mixed` deeper into the domain.

## Bad

```php
<?php

declare(strict_types=1);

class DataProcessor
{
    public function process(mixed $data): mixed  // callers learn nothing
    {
        return $data;
    }

    public function transform(mixed $input, mixed $options): mixed
    {
        // all type information is gone
    }
}

class Cache
{
    public function get(string $key): mixed  // could be anything
    {
        return $this->storage[$key] ?? null;
    }

    public function set(string $key, mixed $value): void
    {
        $this->storage[$key] = $value;
    }
}
```

## Better

```php
<?php

declare(strict_types=1);

class DataProcessor
{
    public function process(array|object $data): array
    {
        return is_object($data) ? get_object_vars($data) : $data;
    }
}

// Type-specific cache avoids mixed entirely
final class UserCache
{
    /** @var array<string, User> */
    private array $store = [];

    public function get(string $id): ?User
    {
        return $this->store[$id] ?? null;
    }

    public function set(string $id, User $user): void
    {
        $this->store[$id] = $user;
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

namespace App\Infrastructure;

// mixed at the boundary — immediately narrowed
final class JsonDecoder
{
    /**
     * Decodes and narrows to a specific shape.
     *
     * @return array<string, mixed>
     * @throws \JsonException
     */
    public function decodeAssoc(string $json): array
    {
        $decoded = json_decode($json, true, 512, JSON_THROW_ON_ERROR);

        if (! is_array($decoded)) {
            throw new \UnexpectedValueException('Expected JSON object, got scalar');
        }

        return $decoded;  // mixed is gone — callers receive array<string, mixed>
    }
}

// Generic template approach for truly polymorphic containers
/**
 * @template T
 */
final class TypedCollection
{
    /** @var list<T> */
    private array $items = [];

    /**
     * @param T $item
     */
    public function add(mixed $item): void  // mixed needed for template satisfaction
    {
        $this->items[] = $item;
    }

    /**
     * @return list<T>
     */
    public function all(): array
    {
        return $this->items;
    }
}

// Concrete specialisation — mixed never escapes to the caller
/**
 * @extends TypedCollection<User>
 */
final class UserCollection extends TypedCollection
{
    public function add(mixed $item): void
    {
        if (! $item instanceof User) {
            throw new \TypeError('Expected User, got ' . get_debug_type($item));
        }
        parent::add($item);
    }
}

// Legitimate mixed at a framework boundary — narrowed immediately
function resolveServiceFromContainer(string $abstract): object
{
    $resolved = app($abstract);  // returns mixed

    if (! is_object($resolved)) {
        throw new \RuntimeException("{$abstract} did not resolve to an object");
    }

    return $resolved;  // object — mixed is gone
}
```

## Exceptions / trade-offs
- **JSON-serialisable values**: `json_encode` legitimately accepts any scalar, array, or `JsonSerializable` — `mixed` is appropriate for the value parameter. Annotate with `@param scalar|array|object|null` to give static analysers more detail.
- **Generic template parameters**: PHPDoc `@template T` requires `mixed` in the native signature of template-satisfying methods. This is an unavoidable limitation of PHP's type system — document the intent clearly.
- **Framework integration points**: Some PSR contracts and framework hooks expose `mixed` (e.g., middleware `$handler`). Accept it at the adapter layer, narrow before passing to your domain.

## Static-analysis notes
PHPStan treats `mixed` as a special type that bypasses many checks — code touching `mixed` values requires explicit narrowing via `is_*`, `instanceof`, or `assert`. At PHPStan level 8+ (`mixed` strictness), operations on `mixed` without narrowing are flagged as errors. Psalm similarly flags `mixed` usage in strict mode. Use `get_debug_type()` (PHP 8.0+) in runtime narrowing error messages for precise type names.

## Version notes
`PHP 8.0+` — `mixed` as a native type declaration. `get_debug_type()` also requires PHP 8.0+.

## Related topics
- [type-union-types.md](type-union-types.md) — narrow specific alternatives instead of mixed
- [type-intersection-types.md](type-intersection-types.md) — compose contracts instead of accepting mixed objects
- [type-parameter-types.md](type-parameter-types.md) — typed parameters eliminate mixed at entry
- [type-return-types.md](type-return-types.md) — typed returns eliminate mixed at exit
