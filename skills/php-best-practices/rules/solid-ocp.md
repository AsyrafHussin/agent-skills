---
title: Open/Closed Principle (OCP)
impact: HIGH
impactDescription: SOLID principles and clean OOP design
tags: php, solid, oop, design-patterns
---

# Open/Closed Principle (OCP)

## Why it matters
Every time you open a class to add a new `if` branch for a new variant, you risk breaking the existing variants. A payment processor with five payment types inline has been modified five times — each change a potential regression. The real cost is not just bugs; it's that your test suite must re-verify unchanged paths every time.

## Rule
Extend where change is expected. Do not pre-abstract imaginary futures. Introduce interfaces and strategy patterns at the boundaries where the domain genuinely varies — not "just in case" you might need a third implementation someday.

## Bad

```php
<?php

declare(strict_types=1);

final class PaymentProcessor
{
    public function process(string $type, float $amount): PaymentResult
    {
        if ($type === 'credit_card') {
            $fee = $amount * 0.029;
            return new PaymentResult($amount + $fee, 'credit_card');
        }

        if ($type === 'paypal') {
            $fee = $amount * 0.035;
            return new PaymentResult($amount + $fee, 'paypal');
        }

        // Adding crypto means modifying this class again
        if ($type === 'crypto') {
            $fee = $amount * 0.01;
            return new PaymentResult($amount + $fee, 'crypto');
        }

        throw new \InvalidArgumentException("Unknown payment type: {$type}");
    }
}
```

## Better

```php
<?php

declare(strict_types=1);

interface PaymentMethod
{
    public function process(Money $amount): PaymentResult;
    public function getName(): string;
}

final class CreditCardPayment implements PaymentMethod
{
    public function __construct(private readonly PaymentGateway $gateway) {}

    public function process(Money $amount): PaymentResult
    {
        return $this->gateway->charge($amount->multiply(1.029));
    }

    public function getName(): string { return 'credit_card'; }
}

final class PaymentProcessor
{
    /** @var array<string, PaymentMethod> */
    private array $methods = [];

    public function register(PaymentMethod $method): void
    {
        $this->methods[$method->getName()] = $method;
    }

    public function process(string $methodName, Money $amount): PaymentResult
    {
        if (!isset($this->methods[$methodName])) {
            throw new \InvalidArgumentException("Unknown payment method: {$methodName}");
        }
        return $this->methods[$methodName]->process($amount);
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

interface PaymentMethod
{
    public function process(Money $amount): PaymentResult;
    public function calculateFee(Money $amount): Money;
    public function getName(): string;
}

final class CreditCardPayment implements PaymentMethod
{
    private const FEE_RATE = 0.029;

    public function __construct(private readonly PaymentGateway $gateway) {}

    public function process(Money $amount): PaymentResult
    {
        return $this->gateway->charge($amount->add($this->calculateFee($amount)));
    }

    public function calculateFee(Money $amount): Money
    {
        return $amount->multiply(self::FEE_RATE);
    }

    public function getName(): string { return 'credit_card'; }
}

final class PayPalPayment implements PaymentMethod
{
    private const FEE_RATE = 0.035;

    public function __construct(private readonly PayPalClient $client) {}

    public function process(Money $amount): PaymentResult
    {
        return $this->client->createPayment($amount->add($this->calculateFee($amount)));
    }

    public function calculateFee(Money $amount): Money
    {
        return $amount->multiply(self::FEE_RATE);
    }

    public function getName(): string { return 'paypal'; }
}

final class CryptoPayment implements PaymentMethod
{
    private const FEE_RATE = 0.01;

    public function __construct(private readonly CryptoGateway $gateway) {}

    public function process(Money $amount): PaymentResult
    {
        return $this->gateway->processPayment($amount->add($this->calculateFee($amount)));
    }

    public function calculateFee(Money $amount): Money
    {
        return $amount->multiply(self::FEE_RATE);
    }

    public function getName(): string { return 'crypto'; }
}

// New payment method? Write a new class. Touch nothing else.
final class PaymentProcessor
{
    /** @var array<string, PaymentMethod> */
    private array $methods = [];

    public function register(PaymentMethod $method): void
    {
        $this->methods[$method->getName()] = $method;
    }

    public function process(string $methodName, Money $amount): PaymentResult
    {
        $method = $this->methods[$methodName]
            ?? throw new UnsupportedPaymentMethodException($methodName);

        return $method->process($amount);
    }
}

// Wiring happens in the composition root, not inside business logic
final class PaymentServiceProvider
{
    public function register(Container $container): void
    {
        $container->singleton(PaymentProcessor::class, static function (Container $c): PaymentProcessor {
            $processor = new PaymentProcessor();
            $processor->register($c->make(CreditCardPayment::class));
            $processor->register($c->make(PayPalPayment::class));
            $processor->register($c->make(CryptoPayment::class));
            return $processor;
        });
    }
}
```

## Strategy pattern — second OCP example

```php
<?php

declare(strict_types=1);

// Discount strategies: add a new one without touching existing code
interface DiscountStrategy
{
    public function calculate(Money $amount): Money;
    public function getDescription(): string;
}

final class PercentageDiscount implements DiscountStrategy
{
    public function __construct(
        private readonly float $percentage,
    ) {}

    public function calculate(Money $amount): Money
    {
        return $amount->multiply($this->percentage);
    }

    public function getDescription(): string
    {
        return sprintf('%d%% off', (int) ($this->percentage * 100));
    }
}

final class FixedAmountDiscount implements DiscountStrategy
{
    public function __construct(
        private readonly Money $discountAmount,
    ) {}

    public function calculate(Money $amount): Money
    {
        return $this->discountAmount->min($amount);
    }

    public function getDescription(): string
    {
        return "{$this->discountAmount->format()} off";
    }
}

final class BuyOneGetOneFreeDiscount implements DiscountStrategy
{
    public function calculate(Money $amount): Money
    {
        return $amount->divide(2);
    }

    public function getDescription(): string
    {
        return 'Buy one get one free';
    }
}

// DiscountCalculator is closed for modification — open to new strategies
final class DiscountCalculator
{
    public function apply(Money $amount, DiscountStrategy $strategy): Money
    {
        return $amount->subtract($strategy->calculate($amount));
    }
}
```

## Exceptions / trade-offs
OCP is not a mandate to wrap everything in interfaces upfront. If you have one payment method and no concrete plan to add another, the interface is premature. Apply OCP at the second variant, not the first. Over-abstracting creates indirection that hurts readability with no payoff. The principle says "closed for modification" — an early-stage class that is modified routinely because the design is still evolving is not a violation.

## Static-analysis notes
PHPStan/Psalm will flag calls to undefined methods on concrete types but won't enforce OCP structurally. Use `@final` or `final` to signal that a class is closed. Architecture rule tools like Deptrac can enforce that high-level modules only reference interfaces, not concrete implementations.

## Related topics
- [solid-srp.md](solid-srp.md) — separate responsibilities first; OCP applies per-responsibility
- [solid-dip.md](solid-dip.md) — depend on the interface, not the concrete implementation
