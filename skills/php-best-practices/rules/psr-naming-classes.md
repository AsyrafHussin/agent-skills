---
title: PSR Class Naming
impact: HIGH
impactDescription: PSR standards and code structure conventions
tags: php, psr, coding-standards
---

# PSR Class Naming

## Why it matters
`UserHelper`, `UserManager`, and `UserUtils` tell you nothing about what the class actually does. When every ambiguous concept gets dumped into a `*Helper` or `*Manager`, those classes grow without bound and become impossible to test in isolation. A name that reveals intent (`UserWelcomeMailer`, `InvoiceLineItemCalculator`) makes the class's single responsibility self-enforcing — it is hard to add unrelated behavior to a class with a precise name.

## Rule
Class names are nouns with intent. The suffix should reflect the role: `Service`, `Repository`, `Handler`, `Builder`, `Factory`, `Validator`, `Notifier`, `Formatter`. Avoid suffix garbage collection: `Helper`, `Util`, `Manager`, `Processor`, `Wrapper` almost always signal that a class does too many things or its purpose is undefined.

## Bad

```php
<?php

declare(strict_types=1);

class UserHelper {}      // what kind of help?
class Util {}            // for what?
class Manager {}         // manages what, exactly?
class GetUser {}         // verb as class name — sounds like a method
class UserProcess {}     // vague action suffix
class DataHandler {}     // 'data' and 'handler' are both meaningless here
```

## Better

```php
<?php

declare(strict_types=1);

// Domain entity — simple noun
final class User {}

// Service — NounVerb or NounService
final class UserService {}
final class OrderShipmentService {}

// Repository — NounRepository
final class UserRepository {}

// Factory — NounFactory
final class OrderFactory {}

// Event — past tense noun
final readonly class UserRegistered
{
    public function __construct(
        public readonly UserId $userId,
        public readonly \DateTimeImmutable $occurredAt,
    ) {}
}

// Exception — DescriptiveException
final class UserNotFoundException extends \RuntimeException {}
final class InsufficientFundsException extends \DomainException {}

// Value Objects — descriptive nouns, immutable, no -VO suffix
final readonly class EmailAddress {}  // not EmailAddressValueObject
final readonly class Money {}         // not MoneyAmount or CurrencyValue
final readonly class DateRange {}     // not DateRangeDto
```

## Best

```php
<?php

declare(strict_types=1);

namespace App\Domain\Order;

// Entity
final class Order
{
    public function __construct(
        private readonly OrderId $id,
        private readonly CustomerId $customerId,
        private OrderStatus $status,
    ) {}

    public function markAsShipped(): void
    {
        $this->status = OrderStatus::Shipped;
    }
}

// Value Object — descriptive noun, immutable
final readonly class OrderId
{
    public function __construct(public readonly string $value)
    {
        if (trim($value) === '') {
            throw new \InvalidArgumentException('OrderId cannot be empty');
        }
    }
}

// Enum — singular noun
enum OrderStatus: string
{
    case Pending  = 'pending';
    case Shipped  = 'shipped';
    case Canceled = 'canceled';
}

// Repository interface — NounRepository, no "I" prefix
interface OrderRepository
{
    public function find(OrderId $id): ?Order;
    public function save(Order $order): void;
}

// Domain event — past tense
final readonly class OrderShipped
{
    public function __construct(
        public readonly OrderId $orderId,
        public readonly \DateTimeImmutable $shippedAt,
    ) {}
}

// Command — imperative verb + noun
final readonly class ShipOrder
{
    public function __construct(
        public readonly OrderId $orderId,
        public readonly string $trackingNumber,
    ) {}
}

// Handler — NounHandler
final class ShipOrderHandler
{
    public function __construct(
        private readonly OrderRepository $repository,
        private readonly OrderShipmentNotifier $notifier,
    ) {}

    public function handle(ShipOrder $command): void
    {
        $order = $this->repository->find($command->orderId)
            ?? throw new OrderNotFoundException($command->orderId);

        $order->markAsShipped();
        $this->repository->save($order);
        $this->notifier->notifyShipped($order);
    }
}
```

## Exceptions / trade-offs
`Controller` (HTTP layer), `Middleware`, `Migration`, and `Seeder` are framework-mandated suffixes — follow the framework convention there. Abstract base classes may use `Abstract` prefix (`AbstractRepository`) if the abstraction is genuinely shared, but prefer composition and interfaces over abstract inheritance. Avoid `Base*` — it says nothing about purpose.

## Static-analysis notes
PHPStan and Psalm do not enforce naming conventions. Use PHP_CodeSniffer with a custom sniff or tools like `phparkitect` to enforce naming rules (e.g., "all classes in `Domain\*\Events` must end in a past-tense word"). Psalm's `@template` annotations work best with precisely named generic classes.

## Related topics
- [psr-naming-methods.md](psr-naming-methods.md) — method naming follows from class intent
- [solid-srp.md](solid-srp.md) — a precise class name enforces single responsibility
