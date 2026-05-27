---
title: Nullsafe Operator
impact: CRITICAL
impactDescription: Modern PHP 8.x language features
tags: php, modern-php, php8
---

# Nullsafe Operator

## Why it matters
Pre-8.0 null-guard pyramids (`if ($a !== null && $a->b !== null && …`) are tedious to write and easy to get wrong — a missing check causes a fatal error in production. The `?->` operator collapses the whole chain into a single readable expression and returns `null` as soon as any link in the chain is null.

## Rule
Use `?->` for optional object traversal chains. Do not use it as a blanket excuse to ignore fundamentally bad nullability design — if an object is always expected to have a sub-object, guarantee that through types, not silent skipping.

## Bad
```php
<?php

declare(strict_types=1);

// Pyramid of doom
$country = null;
if ($user !== null) {
    if ($user->getAddress() !== null) {
        if ($user->getAddress()->getCountry() !== null) {
            $country = $user->getAddress()->getCountry()->getName();
        }
    }
}
```

## Better
```php
<?php

declare(strict_types=1);

// Flat, readable chain — returns null if any link is null
$country = $user?->getAddress()?->getCountry()?->getName();
```

## Best
```php
<?php

declare(strict_types=1);

// Nullsafe chain with a sensible fallback via null coalescing
$countryName = $user?->getAddress()?->getCountry()?->getName() ?? 'Unknown';

// Combined with method calls that themselves return nullable
$formattedCity = $user?->getAddress()?->getCity()?->format() ?? 'N/A';
```

## Exceptions / trade-offs
Overusing `?->` can paper over design flaws. If an `Order` always has a `Customer`, model that as a non-nullable type and let an exception surface when the invariant is violated — silent `null` returns hide bugs. Reserve `?->` for genuinely optional relationships (e.g., a user's optional billing address).

## Common patterns

```php
<?php

declare(strict_types=1);

// Combine with null coalescing for a safe default
$country = $user?->getAddress()?->getCountry()?->getName() ?? 'Unknown';
$managerEmail = $employee?->getDepartment()?->getManager()?->getEmail() ?? null;

// Chain with method calls that mutate optional state
$user?->getPreferences()?->getNotificationChannel()?->send($message);

// First item of an optional collection (null if either is null)
$firstItem = $order?->getItems()[0] ?? null;

// Multiple chains providing a fallback
$contact = $invoice?->getCustomer()?->getPrimaryContact()?->getEmail()
    ?? $invoice?->getCustomer()?->getEmail()
    ?? 'No contact';
```

## When NOT to use nullsafe

```php
<?php

declare(strict_types=1);

// BAD: using ?-> to hide a missing invariant
// An Order always has a Customer — the null comes from bad code, not design
$total = $order?->getCustomer()?->getCreditLimit() ?? 0; // hides the bug

// GOOD: enforce the invariant; let it fail loudly if violated
final class Order
{
    public function __construct(
        private readonly Customer $customer,  // non-nullable — always required
    ) {}

    public function getCustomer(): Customer
    {
        return $this->customer;
    }
}

// Now there is no null to chain through
$total = $order->getCustomer()->getCreditLimit();
```

## Static-analysis notes
PHPStan and Psalm infer nullable types through nullsafe chains and warn when `?->` is applied to a type that is already non-nullable, helping you spot unnecessary noise in the chain.

## Version notes
`PHP 8.0+`

## Related topics
- [type-nullable-types.md](type-nullable-types.md) — model optionality correctly so nullsafe is needed only where it should be
- [modern-match-expression.md](modern-match-expression.md) — companion pattern for exhaustive null/value handling
