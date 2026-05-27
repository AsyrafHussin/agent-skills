---
title: Readonly Properties
impact: CRITICAL
impactDescription: Modern PHP 8.x language features
tags: php, modern-php, php8
---

# Readonly Properties

## Why it matters
Mutable state that should never change after construction is a hidden bug surface. A setter that "should only be called once" will eventually be called twice. Readonly enforces immutability at the language level, not by convention, and makes the contract clear to every reader and tool.

## Rule
Mark any property that must not change after construction as `readonly`. For simple value types and DTOs, this should be the default choice, not an afterthought.

## Bad
```php
<?php

declare(strict_types=1);

final class Money
{
    private int $amount;
    private string $currency;

    public function __construct(int $amount, string $currency)
    {
        $this->amount = $amount;
        $this->currency = $currency;
    }

    // Nothing stops callers from mutating via a setter added later
    public function setAmount(int $amount): void
    {
        $this->amount = $amount;
    }

    public function amount(): int { return $this->amount; }
    public function currency(): string { return $this->currency; }
}
```

## Better
```php
<?php

declare(strict_types=1);

final class Money
{
    private readonly int $amount;
    private readonly string $currency;

    public function __construct(int $amount, string $currency)
    {
        $this->amount = $amount;
        $this->currency = $currency;
    }

    public function amount(): int { return $this->amount; }
    public function currency(): string { return $this->currency; }
}
```

## Best
```php
<?php

declare(strict_types=1);

final class Money
{
    public function __construct(
        public readonly int $amount,
        public readonly string $currency,
    ) {}

    public function add(self $other): self
    {
        assert($this->currency === $other->currency);
        return new self($this->amount + $other->amount, $this->currency);
    }
}
```

## Exceptions / trade-offs
- Properties that need mutation after construction (e.g., an `Order` status that transitions from `pending` → `paid` → `shipped`) must remain mutable.
- Lazy-loaded properties (set on first access) cannot be `readonly` unless initialised in the constructor.
- ORM-managed entities often need mutable properties driven by hydration; apply `readonly` selectively.

## Static-analysis notes
PHPStan and Psalm track `readonly` as write-once. Assigning a `readonly` property outside the constructor (or outside its declaring class for PHP 8.1) is a type error surfaced at static analysis time, not just runtime.

## Version notes
`PHP 8.1+`

## Related topics
- [modern-readonly-classes.md](modern-readonly-classes.md) — apply `readonly` to every property at once with a class modifier
- [modern-constructor-promotion.md](modern-constructor-promotion.md) — combine `readonly` with constructor promotion
