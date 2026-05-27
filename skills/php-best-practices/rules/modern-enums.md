---
title: Enums
impact: CRITICAL
impactDescription: Modern PHP 8.x language features
tags: php, modern-php, php8
---

# Enums

## Why it matters
Class constants typed as `string` or `int` accept any value of that type — passing `'AKTIVE'` instead of `'active'` is a silent runtime bug. Enums create a closed set enforced by the type system: an invalid case literally cannot be constructed, and every match or comparison is checked by static analysis and the runtime.

## Rule
Prefer backed enums over magic strings and class constants for any fixed set of domain states or categories. Use `from()` / `tryFrom()` at the boundary (input parsing) and the enum type everywhere else.

## Bad
```php
<?php

declare(strict_types=1);

final class OrderStatus
{
    public const PENDING  = 'pending';
    public const PAID     = 'paid';
    public const SHIPPED  = 'shipped';
}

// Accepts any string — no compile-time or static-analysis safety
function shipOrder(string $status): void
{
    if ($status === OrderStatus::SHIPPED) { /* ... */ }
}

shipOrder('shiped'); // typo — no error, wrong branch taken
```

## Better
```php
<?php

declare(strict_types=1);

enum OrderStatus
{
    case Pending;
    case Paid;
    case Shipped;
}

// Unit enum: type-safe, but no serialisation value
function shipOrder(OrderStatus $status): void
{
    if ($status === OrderStatus::Shipped) { /* ... */ }
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

// Safe boundary conversion — returns null on unknown value
$status = OrderStatus::tryFrom($request->get('status'));

if ($status === null) {
    throw new \InvalidArgumentException('Unknown order status.');
}

// Everywhere else: the type system prevents invalid values
function shipOrder(OrderStatus $status): void
{
    if ($status === OrderStatus::Shipped) { /* ... */ }
}
```

## Exceptions / trade-offs
- Open-ended or user-defined value sets (e.g., user-created tags, plugin-registered types) are not a fixed set — use a validated string or a domain class instead.
- When the set of valid values is driven entirely by external configuration or a database, enums are the wrong tool.

## Int-backed enums

```php
<?php

declare(strict_types=1);

// Int-backed enum for priority ordering
enum Priority: int
{
    case Low      = 1;
    case Medium   = 2;
    case High     = 3;
    case Critical = 4;

    public function isUrgent(): bool
    {
        return $this->value >= self::High->value;
    }
}

$priority = Priority::High;
if ($priority->isUrgent()) {
    // escalate
}
```

## Enums implementing interfaces

```php
<?php

declare(strict_types=1);

interface HasLabel
{
    public function label(): string;
}

enum PaymentMethod: string implements HasLabel
{
    case CreditCard   = 'credit_card';
    case BankTransfer = 'bank_transfer';
    case PayPal       = 'paypal';

    public function label(): string
    {
        return match($this) {
            self::CreditCard   => 'Credit Card',
            self::BankTransfer => 'Bank Transfer',
            self::PayPal       => 'PayPal',
        };
    }

    public function processingFee(): float
    {
        return match($this) {
            self::CreditCard   => 0.029,
            self::BankTransfer => 0.010,
            self::PayPal       => 0.034,
        };
    }
}

/** @param list<HasLabel> $options */
function renderOptions(array $options): string
{
    return implode(', ', array_map(fn(HasLabel $o) => $o->label(), $options));
}
```

## Enum traits (shared helpers)

```php
<?php

declare(strict_types=1);

// Reusable helper methods for backed enums
trait BackedEnumHelpers
{
    /** @return list<string|int> */
    public static function values(): array
    {
        return array_column(self::cases(), 'value');
    }

    /** @return list<string> */
    public static function names(): array
    {
        return array_column(self::cases(), 'name');
    }
}

enum Role: string
{
    use BackedEnumHelpers;

    case Admin  = 'admin';
    case Editor = 'editor';
    case Viewer = 'viewer';

    public function label(): string
    {
        return match($this) {
            self::Admin  => 'Administrator',
            self::Editor => 'Content Editor',
            self::Viewer => 'Read Only',
        };
    }

    /** @return list<string> */
    public function permissions(): array
    {
        return match($this) {
            self::Admin  => ['create', 'read', 'update', 'delete', 'manage'],
            self::Editor => ['create', 'read', 'update'],
            self::Viewer => ['read'],
        };
    }
}

Role::values(); // ['admin', 'editor', 'viewer']
Role::names();  // ['Admin', 'Editor', 'Viewer']
```

## State-machine transitions

```php
<?php

declare(strict_types=1);

enum OrderStatus: string
{
    case Pending    = 'pending';
    case Processing = 'processing';
    case Shipped    = 'shipped';
    case Delivered  = 'delivered';
    case Cancelled  = 'cancelled';

    /** @return list<self> */
    public function allowedTransitions(): array
    {
        return match($this) {
            self::Pending    => [self::Processing, self::Cancelled],
            self::Processing => [self::Shipped, self::Cancelled],
            self::Shipped    => [self::Delivered],
            self::Delivered,
            self::Cancelled  => [],
        };
    }

    public function canTransitionTo(self $next): bool
    {
        return in_array($next, $this->allowedTransitions(), strict: true);
    }
}

function updateOrderStatus(Order $order, OrderStatus $next): void
{
    if (!$order->status->canTransitionTo($next)) {
        throw new \DomainException(
            "Cannot transition from {$order->status->value} to {$next->value}",
        );
    }
    $order->status = $next;
}
```

## Eloquent / database integration

```php
<?php

declare(strict_types=1);

// Eloquent automatically casts backed enums
class Order extends Model
{
    /** @var array<string, class-string> */
    protected $casts = [
        'status'   => OrderStatus::class,
        'priority' => Priority::class,
    ];
}

// Query with enum value — type safe at the PHP layer
Order::where('status', OrderStatus::Pending->value)->get();

// Laravel validation rule
$request->validate([
    'status' => ['required', new \Illuminate\Validation\Rules\Enum(OrderStatus::class)],
]);
```

## Static-analysis notes
PHPStan and Psalm enforce exhaustive `match` on enum types, flagging unhandled cases. Attempting to pass a plain string where an enum is expected is a type error. `from()` is typed as returning the enum; `tryFrom()` returns `?EnumType`, forcing explicit null handling. PHPStan recognises backed enum `->value` as `string` or `int` depending on the backing type.

## Version notes
`PHP 8.1+`

## Related topics
- [modern-enums-methods.md](modern-enums-methods.md) — add behaviour directly to enums
- [modern-match-expression.md](modern-match-expression.md) — exhaustive matching over enum cases
- [design-value-objects.md](design-value-objects.md) — enums for finite sets; value objects for validated scalars
