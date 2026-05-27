---
title: Value Objects Over Primitives
impact: HIGH
impactDescription: PHP design patterns and value object discipline
tags: php, design-patterns, value-objects
---

# Value Objects Over Primitives

## Why it matters

A `string $email` accepts any string. An `int $userId` accepts any integer — negative, zero, someone else's ID. When primitives stand in for domain concepts, validation is duplicated across every entry point, callers can pass arguments in the wrong order because two parameters share the same type, and invalid state only surfaces at runtime. A value object makes a domain concept a first-class type: validation happens once, at construction, and is then guaranteed for the object's lifetime.

## Rule

Replace bare primitives with small, immutable value objects whenever a primitive represents a domain concept with invariants (valid email, positive ID, non-negative amount). Make every value object `final` and `readonly` by default.

## Bad

```php
<?php

declare(strict_types=1);

// Primitive obsession — no constraints, validation scattered everywhere
class OrderService
{
    public function createOrder(
        int $userId,      // any int? negative IDs valid?
        string $email,    // any string? "not-an-email" accepted?
        float $amount,    // zero? negative?
        string $currency, // "BITCOIN"? "xyz"?
    ): void {
        if ($userId <= 0) {
            throw new \InvalidArgumentException('UserId must be positive');
        }
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new \InvalidArgumentException('Invalid email');
        }
        if ($amount <= 0) {
            throw new \InvalidArgumentException('Amount must be positive');
        }
        // Same validation repeated in every service that touches orders
    }
}

// Same primitive types — caller can swap arguments silently
$service->createOrder(42, 'USD', 0.0, 'user@example.com'); // wrong order, valid PHP
```

## Better

```php
<?php

declare(strict_types=1);

// Simple wrapper objects — validation centralized
final class UserId
{
    public function __construct(
        public readonly int $value,
    ) {
        if ($value <= 0) {
            throw new \InvalidArgumentException("UserId must be positive, got {$value}");
        }
    }
}

final class Email
{
    public readonly string $value;

    public function __construct(string $raw)
    {
        $normalized = strtolower(trim($raw));
        if (!filter_var($normalized, FILTER_VALIDATE_EMAIL)) {
            throw new \InvalidArgumentException("Invalid email: {$raw}");
        }
        $this->value = $normalized;
    }
}

// Service now has typed, self-validating parameters
class OrderService
{
    public function createOrder(UserId $userId, Email $email, float $amount): void
    {
        // No defensive validation needed — types guarantee their own invariants
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

// Full value objects: typed, immutable, with domain behavior
final class UserId
{
    public function __construct(
        public readonly int $value,
    ) {
        if ($value <= 0) {
            throw new \InvalidArgumentException("UserId must be positive, got {$value}");
        }
    }

    public function equals(self $other): bool
    {
        return $this->value === $other->value;
    }

    public function __toString(): string
    {
        return (string) $this->value;
    }
}

final class Email
{
    public readonly string $value;

    public function __construct(string $raw)
    {
        $normalized = strtolower(trim($raw));
        if (!filter_var($normalized, FILTER_VALIDATE_EMAIL)) {
            throw new \InvalidArgumentException("Invalid email: {$raw}");
        }
        $this->value = $normalized;
    }

    public function domain(): string
    {
        return substr($this->value, (int) strpos($this->value, '@') + 1);
    }

    public function equals(self $other): bool
    {
        return $this->value === $other->value;
    }
}

final class Money
{
    public function __construct(
        public readonly int $amountCents,    // always in minor units
        public readonly Currency $currency,
    ) {
        if ($this->amountCents < 0) {
            throw new \InvalidArgumentException('Money cannot be negative');
        }
    }

    public function add(self $other): self
    {
        if ($this->currency !== $other->currency) {
            throw new \LogicException('Cannot add money in different currencies');
        }
        return new self($this->amountCents + $other->amountCents, $this->currency);
    }

    public function isZero(): bool
    {
        return $this->amountCents === 0;
    }
}

enum Currency: string
{
    case USD = 'USD';
    case EUR = 'EUR';
    case GBP = 'GBP';
}

enum OrderStatus: string
{
    case Pending   = 'pending';
    case Paid      = 'paid';
    case Shipped   = 'shipped';
    case Cancelled = 'cancelled';
}

// Service: fully typed, zero defensive validation
final class OrderService
{
    public function createOrder(
        UserId $userId,
        Email $email,
        Money $amount,
        OrderStatus $status = OrderStatus::Pending,
    ): Order {
        return new Order($userId, $email, $amount, $status);
    }
}

// Impossible to swap arguments — every parameter has a distinct type
$service->createOrder(
    userId: new UserId(42),
    email:  new Email('user@example.com'),
    amount: new Money(9999, Currency::USD),
);
```

## Exceptions / trade-offs

- **Simple scalars with no invariants** (a raw log message, a free-form description field) do not need wrapping. Not every `string` is a value object candidate.
- **DTO-style transport objects** (API request/response bodies) can stay as typed arrays or simple data classes when they cross system boundaries and don't carry invariants.
- **Third-party library APIs** may require primitive arguments — unwrap at the boundary, not in your domain logic.

A useful test: if two parameters share the same primitive type and could accidentally be swapped, that is a signal to introduce a value object.

## Designing a value object

A good value object is `final`, `readonly`, validates at construction, has an `equals()` method, and can carry domain behavior:

```php
<?php

declare(strict_types=1);

/**
 * Immutable value object. Two Coordinates are equal when lat+lng match.
 */
final class Coordinates
{
    public function __construct(
        public readonly float $latitude,
        public readonly float $longitude,
    ) {
        if ($latitude < -90.0 || $latitude > 90.0) {
            throw new \InvalidArgumentException("Latitude {$latitude} out of range [-90, 90]");
        }
        if ($longitude < -180.0 || $longitude > 180.0) {
            throw new \InvalidArgumentException("Longitude {$longitude} out of range [-180, 180]");
        }
    }

    public function equals(self $other): bool
    {
        return $this->latitude === $other->latitude
            && $this->longitude === $other->longitude;
    }

    public function distanceKmTo(self $other): float
    {
        // Haversine formula — behaviour lives with data
        $R    = 6371.0;
        $dLat = deg2rad($other->latitude  - $this->latitude);
        $dLng = deg2rad($other->longitude - $this->longitude);
        $a    = sin($dLat / 2) ** 2
            + cos(deg2rad($this->latitude)) * cos(deg2rad($other->latitude)) * sin($dLng / 2) ** 2;
        return $R * 2 * atan2(sqrt($a), sqrt(1 - $a));
    }

    public function __toString(): string
    {
        return "{$this->latitude},{$this->longitude}";
    }
}
```

## When to create a value object

| Primitive | Value Object | Reason |
|-----------|-------------|--------|
| `string $email` | `Email` | Validation, normalization, `domain()` method |
| `int $userId` | `UserId` | Prevents mixing with other IDs; `positive-int` constraint |
| `float $amount` + `string $currency` | `Money` | Prevents currency mismatch; `add()` / `multiply()` behavior |
| `string $slug` | `Slug` | URL-safe validation, generation |
| `string $uuid` | `Uuid` | Format validation, version detection |
| `string $phone` | `PhoneNumber` | E.164 normalization, country code access |
| `float $lat` + `float $lng` | `Coordinates` | Range validation, `distanceKmTo()` behavior |

## Static-analysis notes

- `final readonly` value objects give PHPStan / Psalm full type inference — no `mixed` residue.
- PHPStan `@param positive-int $value` or `@return non-empty-string` can annotate interim cases before a full value object is extracted.
- PHPStan will correctly track `UserId` vs `ProductId` as distinct types and flag accidental swaps.
- `equals()` methods on value objects are trivially testable and PHPStan can verify the comparison type.

## Related topics

- [design-no-magic.md](design-no-magic.md) — avoid attribute bags that defeat value object type safety
- [modern-enums.md](modern-enums.md) — use enums instead of string constants for domain states
- [type-property-types.md](type-property-types.md) — type every property in value objects
- [modern-readonly-classes.md](modern-readonly-classes.md) — readonly classes as a shorthand for simple value objects (PHP 8.2+)
- [phpstan-phpdoc.md](phpstan-phpdoc.md) — annotate interim types before extracting full value objects
