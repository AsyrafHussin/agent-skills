---
title: Liskov Substitution Principle (LSP)
impact: HIGH
impactDescription: SOLID principles and clean OOP design
tags: php, solid, oop, design-patterns
---

# Liskov Substitution Principle (LSP)

## Why it matters
A subtype that silently changes the behavior of its parent breaks every caller that was written against the parent's contract — even if PHP's type system says nothing is wrong. The classic `Square extends Rectangle` example is not just an academic puzzle: it represents any subclass that narrows preconditions, widens postconditions, or throws exceptions where the parent would not. These are runtime surprises that static types cannot catch.

## Rule
Substitutable behavior matters more than matching signatures. A subtype that violates the preconditions or postconditions of its parent is not a valid substitution, even if PHP accepts it. If you cannot substitute a subtype everywhere the parent is used without changing the correctness of the program, the inheritance hierarchy is wrong.

## Bad

```php
<?php

declare(strict_types=1);

class Rectangle
{
    public function __construct(
        protected int $width,
        protected int $height,
    ) {}

    public function setWidth(int $width): void { $this->width = $width; }
    public function setHeight(int $height): void { $this->height = $height; }
    public function area(): int { return $this->width * $this->height; }
}

// Violates LSP: mutating width silently changes height, breaking Rectangle's contract
class Square extends Rectangle
{
    public function setWidth(int $width): void
    {
        $this->width = $width;
        $this->height = $width; // caller does not expect this side-effect
    }

    public function setHeight(int $height): void
    {
        $this->width = $height;
        $this->height = $height;
    }
}

function resizeAndMeasure(Rectangle $rect): int
{
    $rect->setWidth(5);
    $rect->setHeight(10);
    return $rect->area(); // expects 50; gets 100 if $rect is a Square
}
```

## Better

```php
<?php

declare(strict_types=1);

interface Shape
{
    public function area(): int;
}

final readonly class Rectangle implements Shape
{
    public function __construct(
        private int $width,
        private int $height,
    ) {}

    public function area(): int { return $this->width * $this->height; }
}

final readonly class Square implements Shape
{
    public function __construct(private int $side) {}

    public function area(): int { return $this->side ** 2; }
}

// Both satisfy the Shape contract — no surprises
function totalArea(Shape ...$shapes): int
{
    return array_sum(array_map(fn(Shape $s) => $s->area(), $shapes));
}
```

## Best

```php
<?php

declare(strict_types=1);

// Interface contract includes behavioral guarantees, not just signatures
interface PaymentGateway
{
    /**
     * @throws InsufficientFundsException
     * @throws PaymentDeclinedException
     * Never returns a result with status Pending — it is always terminal.
     */
    public function charge(Money $amount, PaymentMethod $method): PaymentResult;

    /**
     * Refund must return the same currency as the original payment.
     */
    public function refund(PaymentId $paymentId, Money $amount): RefundResult;
}

final class StripeGateway implements PaymentGateway
{
    public function __construct(private readonly \Stripe\StripeClient $stripe) {}

    public function charge(Money $amount, PaymentMethod $method): PaymentResult
    {
        try {
            $charge = $this->stripe->charges->create([
                'amount'   => $amount->cents(),
                'currency' => $amount->currency(),
            ]);
            return PaymentResult::completed(new PaymentId($charge->id), $amount);
        } catch (\Stripe\Exception\CardException $e) {
            throw match ($e->getStripeCode()) {
                'insufficient_funds' => new InsufficientFundsException($e->getMessage()),
                default              => new PaymentDeclinedException($e->getMessage()),
            };
        }
    }

    public function refund(PaymentId $paymentId, Money $amount): RefundResult
    {
        // Returns same currency as documented in the contract
        $refund = $this->stripe->refunds->create([
            'charge' => $paymentId->value,
            'amount' => $amount->cents(),
        ]);

        return new RefundResult(
            id: new RefundId($refund->id),
            amount: $amount,
        );
    }
}

final class PayPalGateway implements PaymentGateway
{
    public function __construct(private readonly PayPalClient $client) {}

    public function charge(Money $amount, PaymentMethod $method): PaymentResult
    {
        // Different implementation, identical behavioral contract:
        // throws the same exception types under the same conditions,
        // never returns Pending.
        $response = $this->client->createOrder($amount->toPayPalArray());

        return match ($response->status) {
            'COMPLETED' => PaymentResult::completed(new PaymentId($response->id), $amount),
            'DECLINED'  => throw new PaymentDeclinedException($response->message),
            default     => throw new PaymentDeclinedException('Unexpected PayPal status'),
        };
    }

    public function refund(PaymentId $paymentId, Money $amount): RefundResult
    {
        // Same contract honored: returns RefundResult in original currency
        $response = $this->client->createRefund($paymentId->value, $amount->toPayPalArray());

        return new RefundResult(
            id: new RefundId($response->id),
            amount: $amount,
        );
    }
}

// ISP in action: birds split on actual capability, not taxonomy
interface FlyingBird
{
    public function fly(): void;
}

interface SwimmingBird
{
    public function swim(): void;
}

final class Sparrow implements FlyingBird
{
    public function fly(): void { /* flaps wings */ }
}

final class Penguin implements SwimmingBird
{
    public function swim(): void { /* dives */ }
}
```

## Exceptions / trade-offs
Readonly value objects (`readonly class Rectangle`) make LSP violations structurally impossible for mutation-based contracts, because there is nothing to mutate. Prefer immutable types when the domain allows it. Throwing additional, more-specific exceptions in a subtype is acceptable if the parent's documented exceptions are a superset of what callers must handle.

## Static-analysis notes
PHPStan level 6+ and Psalm will catch return type narrowing violations and missing method implementations. They will not catch behavioral violations (e.g., a method that silently changes unrelated state). Document postconditions in docblocks and use `@psalm-assert` / `@phpstan-assert` to surface them to static analyzers.

## Related topics
- [solid-isp.md](solid-isp.md) — splitting fat interfaces prevents forcing implementations that violate postconditions
- [solid-ocp.md](solid-ocp.md) — strategy pattern implementations must also honor LSP
