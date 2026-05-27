---
title: Constructor Property Promotion
impact: CRITICAL
impactDescription: Modern PHP 8.x language features
tags: php, modern-php, php8
---

# Constructor Property Promotion

## Why it matters
Manually declaring properties, then re-listing them as constructor parameters, then assigning them is mechanical repetition. That boilerplate drifts — someone adds a parameter but forgets the property declaration, or renames one place but not another. Promotion eliminates the surface area for these mistakes by collapsing all three steps into one canonical location.

## Rule
Use constructor property promotion for dependency injection and DTOs. Skip it only when the constructor body contains significant logic that would become harder to read alongside the promoted declarations.

## Bad
```php
<?php

declare(strict_types=1);

class UserService
{
    private UserRepository $repository;
    private EventDispatcher $dispatcher;
    private LoggerInterface $logger;

    public function __construct(
        UserRepository $repository,
        EventDispatcher $dispatcher,
        LoggerInterface $logger,
    ) {
        $this->repository = $repository;
        $this->dispatcher = $dispatcher;
        $this->logger = $logger;
    }
}
```

## Better
```php
<?php

declare(strict_types=1);

class UserService
{
    public function __construct(
        private UserRepository $repository,
        private EventDispatcher $dispatcher,
        private LoggerInterface $logger,
    ) {}
}
```

## Best
```php
<?php

declare(strict_types=1);

final class CreateUserCommand
{
    public function __construct(
        public readonly string $email,
        public readonly string $name,
        public readonly string $role = 'user',
    ) {}
}
```

## Exceptions / trade-offs
- When the constructor body has non-trivial logic (validation, transformation, conditional branching), keep traditional declarations so the assignments are not buried among the promoted parameter list.
- Non-promoted properties (e.g., lazily initialised, computed in the body) can coexist, but more than one or two breaks the clarity benefit.

## With validation in the constructor

```php
<?php

declare(strict_types=1);

// Value objects: validate the invariant directly in the promoted constructor
final class Email
{
    public function __construct(
        public readonly string $value,
    ) {
        if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
            throw new \InvalidArgumentException("Invalid email: {$value}");
        }
    }
}

final class Money
{
    public function __construct(
        public readonly int $amountCents,
        public readonly string $currency = 'USD',
    ) {
        if ($amountCents < 0) {
            throw new \InvalidArgumentException('Amount cannot be negative');
        }
        if (strlen($currency) !== 3) {
            throw new \InvalidArgumentException('Currency must be 3-letter ISO code');
        }
    }
}
```

## Mixed promoted and non-promoted

```php
<?php

declare(strict_types=1);

// Computed properties cannot be promoted — mix when needed
final class Order
{
    private string $orderNumber;

    public function __construct(
        public readonly OrderId $id,
        /** @var list<OrderLine> */
        public readonly array $lines,
        public readonly \DateTimeImmutable $createdAt,
    ) {
        // Computed property: cannot be promoted
        $this->orderNumber = sprintf(
            'ORD-%s-%s',
            $createdAt->format('Ymd'),
            strtoupper(substr($id->value, 0, 8)),
        );
    }

    public function getOrderNumber(): string
    {
        return $this->orderNumber;
    }
}
```

## Visibility combinations

```php
<?php

declare(strict_types=1);

final class Entity
{
    public function __construct(
        public readonly string $id,        // public, immutable
        protected string $name,            // protected, mutable
        private string $internalState,     // private, mutable
        private readonly array $metadata,  // private, immutable
    ) {}
}
```

## Static-analysis notes
PHPStan and Psalm infer the property type directly from the promoted parameter type, including nullability. No separate `@var` annotation is required or useful. Promoted `readonly` properties get `@readonly` semantics automatically in PHPStan's analysis.

## Version notes
`PHP 8.0+` (promotion). `readonly` modifier on promoted properties: `PHP 8.1+`.

## Related topics
- [modern-readonly-properties.md](modern-readonly-properties.md) — add `readonly` to promoted properties for immutability
- [modern-readonly-classes.md](modern-readonly-classes.md) — make every property readonly at the class level
- [design-value-objects.md](design-value-objects.md) — validation in promoted constructors for value objects
