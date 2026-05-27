---
title: Property Type Declarations
impact: CRITICAL
impactDescription: Strong type safety across all PHP code
tags: php, types, strict-types, type-safety
---

# Property Type Declarations

## Why it matters
An untyped property is an uncontrolled mutation surface: any assignment can change it to any value at any point in the object's lifetime. Typed properties enforce class invariants at the VM level — the engine throws a `TypeError` before corrupt state reaches any method. `readonly` goes further by making the property assignment-once, eliminating an entire class of accidental mutation bugs.

## Rule
Every class property must carry a native type declaration. Prefer `readonly` for any property whose value is set once at construction time and must not change.

## Bad

```php
<?php

declare(strict_types=1);

class Product
{
    private $id;
    private $name;
    private $price;
    private $categories;
    private $createdAt;

    public function __construct($id, $name, $price)
    {
        $this->id = $id;         // could be anything
        $this->name = $name;
        $this->price = $price;
        $this->categories = [];
    }
}
```

## Better

```php
<?php

declare(strict_types=1);

class Product
{
    private int $id;
    private string $name;
    private float $price;
    /** @var list<string> */
    private array $categories = [];
    private ?string $description = null;
    private \DateTimeImmutable $createdAt;

    public function __construct(int $id, string $name, float $price)
    {
        $this->id = $id;
        $this->name = $name;
        $this->price = $price;
        $this->createdAt = new \DateTimeImmutable();
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

namespace App\Catalogue;

use App\ValueObjects\Money;
use App\ValueObjects\Slug;

final class Product
{
    /** @var list<string> */
    private array $categoryIds = [];

    public function __construct(
        public readonly int $id,
        public readonly string $name,
        public readonly Slug $slug,
        public readonly Money $price,
        private \DateTimeImmutable $updatedAt = new \DateTimeImmutable(),
    ) {}

    public function addCategory(string $categoryId): void
    {
        $this->categoryIds[] = $categoryId;
        $this->updatedAt = new \DateTimeImmutable();
    }

    /** @return list<string> */
    public function categoryIds(): array
    {
        return $this->categoryIds;
    }

    public function updatedAt(): \DateTimeImmutable
    {
        return $this->updatedAt;
    }
}
```

## Exceptions / trade-offs
- **Lazy-initialised properties**: When a property is set after construction (e.g., in a setter or factory step), `readonly` is not appropriate — declare the type without `readonly` and accept that mutation is intentional.
- **Generic collections**: `array` is the only native option for collections; annotate with PHPDoc `@var list<T>` or `@var array<K, V>` for static-analysis precision.
- **ORM-mapped entities**: Some ORMs (Doctrine, Eloquent) hydrate properties via reflection after construction. Use typed properties without `readonly`; the ORM will still catch type violations on flush.
- **`readonly` classes (PHP 8.2+)**: Marking an entire class `readonly` applies `readonly` to all promoted properties automatically — prefer this for pure value objects.

## Static-analysis notes
PHPStan and Psalm track property types through assignments and flag mismatches. `readonly` properties are additionally flagged if assigned more than once. PhpStorm highlights uninitialized typed properties when no default value or constructor assignment is present.

## Version notes
`PHP 7.4+` — typed properties. `PHP 8.1+` — `readonly` properties. `PHP 8.2+` — `readonly` classes.

## Related topics
- [type-strict-mode.md](type-strict-mode.md) — strict mode applies to property assignments too
- [type-parameter-types.md](type-parameter-types.md) — constructor parameter types feed property types
- [type-nullable-types.md](type-nullable-types.md) — when a property may legitimately be null
- [type-union-types.md](type-union-types.md) — when a property holds one of several concrete types
