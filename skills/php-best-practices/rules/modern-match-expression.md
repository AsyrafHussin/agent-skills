---
title: Match Expression
impact: CRITICAL
impactDescription: Modern PHP 8.x language features
tags: php, modern-php, php8
---

# Match Expression

## Why it matters
`switch` has two traps: loose comparison (`==`) silently coerces types, and fall-through means a missing `break` delivers the wrong branch without any error. `match` uses strict comparison, always returns a value, and throws `UnhandledMatchError` on a missing arm — bugs surface immediately instead of silently misbehaving.

## Rule
Prefer `match` over `switch` for expression-oriented branching. When matching an enum, omit `default` so `UnhandledMatchError` fires on unhandled cases rather than silently using a fallback.

## Bad
```php
<?php

declare(strict_types=1);

function label(string $status): string
{
    switch ($status) {
        case 'active':
            $label = 'Active';
            break;
        case 'inactive':
            $label = 'Inactive';
            // missing break — falls through to default silently
        default:
            $label = 'Unknown';
    }
    return $label;
}
```

## Better
```php
<?php

declare(strict_types=1);

function label(string $status): string
{
    return match ($status) {
        'active'   => 'Active',
        'inactive' => 'Inactive',
        default    => 'Unknown',
    };
}
```

## Best
```php
<?php

declare(strict_types=1);

enum OrderStatus: string
{
    case Pending  = 'pending';
    case Paid     = 'paid';
    case Shipped  = 'shipped';
}

// No default: UnhandledMatchError fires if a new case is added but not handled here
function label(OrderStatus $status): string
{
    return match ($status) {
        OrderStatus::Pending  => 'Awaiting payment',
        OrderStatus::Paid     => 'Payment received',
        OrderStatus::Shipped  => 'On its way',
    };
}

// Enum + match returning a value object — exhaustiveness guarantees every case has a fee
enum ShippingMethod: string
{
    case Standard  = 'standard';
    case Express   = 'express';
    case Overnight = 'overnight';
    case Pickup    = 'pickup';
}

final class OrderProcessor
{
    public function calculateShippingFee(Order $order): Money
    {
        $baseRate = match ($order->shippingMethod) {
            ShippingMethod::Standard  => new Money(599,  Currency::USD),
            ShippingMethod::Express   => new Money(1299, Currency::USD),
            ShippingMethod::Overnight => new Money(2499, Currency::USD),
            ShippingMethod::Pickup    => new Money(0,    Currency::USD),
            // No default — adding a new ShippingMethod case without updating here
            // immediately produces UnhandledMatchError, not a silent wrong fee
        };

        return $order->isHeavy()
            ? $baseRate->multiply(1.5)
            : $baseRate;
    }
}
```

## Exceptions / trade-offs
- When genuine fall-through is needed (multiple inputs share one result), `match` supports comma-separated arms: `'a', 'b' => 'result'` — this is not a reason to keep `switch`.
- Complex compound conditions that cannot be expressed as simple value equality (e.g., range checks, regex, multiple variables) are better served by `if`/`elseif` or a strategy pattern.

## Multiple conditions per arm and `match(true)`

```php
<?php

declare(strict_types=1);

// Multiple values in one arm
function discount(string $customerType): float
{
    return match ($customerType) {
        'premium', 'vip', 'gold' => 0.20,
        'silver', 'regular'      => 0.10,
        'new'                    => 0.05,
        default                  => 0.0,
    };
}

// match(true) for range-based conditions
function httpCategory(int $code): string
{
    return match (true) {
        $code >= 200 && $code < 300 => 'Success',
        $code >= 300 && $code < 400 => 'Redirection',
        $code >= 400 && $code < 500 => 'Client Error',
        $code >= 500                => 'Server Error',
        default                     => 'Informational',
    };
}

// Strict comparison: '1' === 1 is false — match never coerces
$value = '1';
$result = match ($value) {
    1   => 'integer one', // will NOT match '1'
    '1' => 'string one',  // matches
    default => 'other',
};
```

## Static-analysis notes
PHPStan and Psalm verify exhaustiveness when `match` operates on an enum type with no `default` arm. Adding a new enum case without updating every `match` site becomes a static analysis error.

## Version notes
`PHP 8.0+`

## Related topics
- [modern-enums.md](modern-enums.md) — backed enums as safe match subjects
- [modern-enums-methods.md](modern-enums-methods.md) — exhaustive match inside enum methods
