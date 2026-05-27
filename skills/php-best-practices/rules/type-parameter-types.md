---
title: Parameter Type Declarations
impact: CRITICAL
impactDescription: Strong type safety across all PHP code
tags: php, types, strict-types, type-safety
---

# Parameter Type Declarations

## Why it matters
Untyped parameters are an open door: any caller can pass an integer where an object is expected, an array where a string is required, or `null` where neither is valid. The bug surfaces deep inside the method body — or worse, in a downstream system — rather than at the call site where it was introduced. Typed parameters shift that error boundary to the earliest possible moment.

## Rule
Every function and method parameter must carry a native type declaration. Use PHPDoc (`@param`) only when the native type system cannot express the constraint (e.g., generic array shapes).

## Bad

```php
<?php

declare(strict_types=1);

class OrderService
{
    // $user could be an ID, an array, an object — impossible to know
    public function createOrder($user, $items, $discount)
    {
        // type assumptions buried inside the method
    }

    public function processPayment($amount, $method)
    {
        // silent failure when wrong types arrive
    }
}
```

## Better

```php
<?php

declare(strict_types=1);

class OrderService
{
    /**
     * @param list<array{sku: string, qty: int}> $items
     */
    public function createOrder(
        User $user,
        array $items,
        float $discountPercentage = 0.0,
    ): Order {
        $order = new Order($user);

        foreach ($items as $item) {
            $order->addItem($item['sku'], $item['qty']);
        }

        $order->applyDiscount($discountPercentage);

        return $order;
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

namespace App\Commerce;

use App\Models\Order;
use App\Models\User;
use App\ValueObjects\Discount;
use App\ValueObjects\OrderItem;

final class OrderService
{
    public function __construct(
        private readonly OrderRepository $orders,
        private readonly PaymentGateway $gateway,
    ) {}

    /**
     * @param list<OrderItem> $items
     */
    public function createOrder(
        User $user,
        array $items,
        Discount $discount,
    ): Order {
        $order = Order::for($user);

        foreach ($items as $item) {
            $order->add($item);
        }

        $order->apply($discount);

        return $this->orders->save($order);
    }

    public function processPayment(
        Order $order,
        PaymentMethod $method,
    ): PaymentResult {
        return $this->gateway->charge($order->total(), $method);
    }
}
```

## Exceptions / trade-offs
- **Callback parameters**: When passing a `callable`, its internal signature cannot be expressed natively — use `\Closure` for strict callables or PHPDoc `@param callable(Foo): Bar` for shape documentation.
- **Generic collections**: Native PHP cannot express `array<string, User>`; use PHPDoc `@param` alongside the native `array` type declaration.
- **Framework magic**: Some framework base classes inject dependencies through untyped magic methods (e.g., older Laravel traits). Add the type where you control the code; leave framework internals alone.

## Static-analysis notes
PHPStan (level 5+) and Psalm flag untyped parameters as errors. With `strict_types=1`, wrong-type call sites are also caught at runtime. PhpStorm provides full parameter-hint autocompletion only when types are declared. Use `@param` PHPDoc for array shapes that PHPStan's `array<K, V>` syntax can verify.

## Version notes
`PHP 7.0+` — scalar type declarations (`int`, `string`, `float`, `bool`) require PHP 7.0.

## Related topics
- [type-strict-mode.md](type-strict-mode.md) — strict mode enforces parameter types at runtime
- [type-return-types.md](type-return-types.md) — pair every typed parameter with a typed return
- [type-nullable-types.md](type-nullable-types.md) — when null is a valid parameter value
- [type-union-types.md](type-union-types.md) — when a parameter accepts multiple concrete types
