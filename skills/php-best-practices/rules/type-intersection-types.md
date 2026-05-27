---
title: Intersection Types
impact: CRITICAL
impactDescription: Strong type safety across all PHP code
tags: php, types, strict-types, type-safety
---

# Intersection Types

## Why it matters
Without intersection types, a method that requires an object to satisfy two interfaces must choose one as the parameter type and either document the other in a comment (unenforced) or add a runtime `instanceof` guard (deferred error). Both approaches let the wrong object past the call site. Intersection types close that gap at the language level.

## Rule
Use an intersection type (`A&B`) when a parameter or return value must simultaneously satisfy multiple contracts. Do not create a new combined interface just to avoid the syntax — keep contracts small and compose them at the call site.

## Bad

```php
<?php

declare(strict_types=1);

interface Cacheable
{
    public function getCacheKey(): string;
}

interface Serializable
{
    public function serialize(): string;
}

class CacheStore
{
    /**
     * @param Cacheable&Serializable $item   ← docblock only, not enforced
     */
    public function store($item): void  // no native type — anything passes
    {
        $key  = $item->getCacheKey();
        $data = $item->serialize();
        $this->backend->set($key, $data);
    }
}
```

## Better

```php
<?php

declare(strict_types=1);

interface Cacheable
{
    public function getCacheKey(): string;
    public function cacheTtl(): int;
}

interface Serializable
{
    public function serialize(): string;
    public function unserialize(string $data): void;
}

class CacheStore
{
    // enforced at runtime — both interfaces required
    public function store(Cacheable&Serializable $item): void
    {
        $this->backend->set(
            $item->getCacheKey(),
            $item->serialize(),
            $item->cacheTtl(),
        );
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

namespace App\Cache;

interface Cacheable
{
    public function cacheKey(): string;
    public function cacheTtl(): int;
}

interface JsonSerializable
{
    public function toJson(): string;
    public static function fromJson(string $json): static;
}

final class RedisStore
{
    public function __construct(
        private readonly \Redis $client,
    ) {}

    public function put(Cacheable&JsonSerializable $item): void
    {
        $this->client->setex(
            $item->cacheKey(),
            $item->cacheTtl(),
            $item->toJson(),
        );
    }

    /**
     * @template T of Cacheable&JsonSerializable
     * @param T $prototype
     * @return T|null
     */
    public function get(string $key, Cacheable&JsonSerializable $prototype): Cacheable&JsonSerializable|null
    {
        $raw = $this->client->get($key);

        return $raw !== false ? $prototype::fromJson($raw) : null;
    }
}

// Concrete implementation satisfying both interfaces
final class UserSession implements Cacheable, JsonSerializable
{
    public function __construct(
        private readonly int $userId,
        private readonly string $token,
    ) {}

    public function cacheKey(): string
    {
        return "session:{$this->userId}";
    }

    public function cacheTtl(): int
    {
        return 3600;
    }

    public function toJson(): string
    {
        return json_encode(['userId' => $this->userId, 'token' => $this->token], JSON_THROW_ON_ERROR);
    }

    public static function fromJson(string $json): static
    {
        $data = json_decode($json, true, 512, JSON_THROW_ON_ERROR);
        return new static($data['userId'], $data['token']);
    }
}
```

## Exceptions / trade-offs
- **Do not merge interfaces**: Resist creating `interface CacheableAndSerializable extends Cacheable, Serializable {}` solely to avoid intersection syntax — it forces classes to implement a combined interface instead of two focused ones.
- **Intersection with classes**: PHP requires all members of an intersection type to be interfaces or abstract classes — you cannot intersect concrete classes.
- **Return type intersections**: Returning `A&B` from a method is valid and useful (e.g., factory methods), but the concrete class must implement both interfaces.
- **DNF types (PHP 8.2+)**: Disjunctive Normal Form allows combining unions and intersections: `(A&B)|null`. Use for nullable intersection parameters.

## Static-analysis notes
PHPStan and Psalm fully support intersection types, including `@template T of A&B` for generic intersection constraints. Both tools verify that all interface members are accessible and that concrete types satisfy the intersection at assignment sites.

## Version notes
`PHP 8.1+` — native intersection types. `PHP 8.2+` — DNF types (`(A&B)|null`).

## Related topics
- [type-union-types.md](type-union-types.md) — alternative states vs. combined contracts
- [type-parameter-types.md](type-parameter-types.md) — use intersection at parameter boundaries
- [type-return-types.md](type-return-types.md) — return intersection types from factory methods
