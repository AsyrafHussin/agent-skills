---
title: Dependency Inversion Principle (DIP)
impact: HIGH
impactDescription: SOLID principles and clean OOP design
tags: php, solid, oop, design-patterns
---

# Dependency Inversion Principle (DIP)

## Why it matters
High-level business logic that instantiates `new MySqlDatabase(...)` or `new SmtpMailer(...)` internally is untestable without a real database and a real mail server. It is also immovable — switching from MySQL to PostgreSQL requires editing the class that should know nothing about storage. The coupling is invisible until it bites you in a test suite that takes ten minutes because every test hits the database.

## Rule
Depend on abstractions at meaningful boundaries, not everywhere by ritual. High-level modules (business logic, application services) must not depend on concrete low-level implementations (databases, mailers, HTTP clients). Both should depend on interfaces defined by the high-level module. Do not create interfaces for trivial utilities where there is genuinely only one implementation and no testing need.

## Bad

```php
<?php

declare(strict_types=1);

final class OrderService
{
    private MySqlDatabase $database;
    private SmtpMailer $mailer;
    private StripePayment $payment;

    public function __construct()
    {
        // Hard-coded infrastructure — untestable, unswappable
        $this->database = new MySqlDatabase('localhost', 'shop', 'user', 'pass');
        $this->mailer   = new SmtpMailer('smtp.gmail.com', 587);
        $this->payment  = new StripePayment('sk_live_...');
    }

    public function createOrder(array $data): Order
    {
        $order = new Order($data);
        $this->payment->charge($order->getTotal());
        $this->database->insert('orders', $order->toArray());
        $this->mailer->send($order->getCustomerEmail(), 'Confirmed', '...');
        return $order;
    }
}
```

## Better

```php
<?php

declare(strict_types=1);

// Abstractions defined in the domain/application layer
interface OrderRepository
{
    public function save(Order $order): void;
}

interface PaymentGateway
{
    public function charge(Money $amount, PaymentMethod $method): PaymentResult;
}

interface Mailer
{
    public function send(string $to, string $subject, string $body): void;
}

// High-level module depends only on interfaces
final class OrderService
{
    public function __construct(
        private readonly OrderRepository $repository,
        private readonly PaymentGateway $payment,
        private readonly Mailer $mailer,
    ) {}

    public function createOrder(CreateOrderCommand $command): Order
    {
        $order  = Order::create($command->customerId, $command->items);
        $result = $this->payment->charge($order->total(), $command->paymentMethod);

        if (!$result->isSuccessful()) {
            throw new PaymentFailedException($result->error());
        }

        $order->markAsPaid($result->transactionId());
        $this->repository->save($order);
        $this->mailer->send($command->email, 'Order confirmed', '...');

        return $order;
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

// Domain interfaces — owned by the application layer
interface OrderRepository
{
    public function save(Order $order): void;
    public function find(OrderId $id): ?Order;
}

interface PaymentGateway
{
    /** @throws PaymentFailedException */
    public function charge(Money $amount, PaymentMethod $method): PaymentResult;
}

interface OrderNotifier
{
    public function confirmationSent(Order $order): void;
}

// Application service: pure orchestration, zero infrastructure knowledge
final class OrderService
{
    public function __construct(
        private readonly OrderRepository $repository,
        private readonly PaymentGateway $payment,
        private readonly OrderNotifier $notifier,
    ) {}

    public function createOrder(CreateOrderCommand $command): Order
    {
        $order  = Order::create($command->customerId, $command->items);
        $result = $this->payment->charge($order->total(), $command->paymentMethod);
        $order->markAsPaid($result->transactionId());
        $this->repository->save($order);
        $this->notifier->confirmationSent($order);
        return $order;
    }
}

// Infrastructure: implements the interfaces
final class DoctrineOrderRepository implements OrderRepository
{
    public function __construct(private readonly EntityManagerInterface $em) {}

    public function save(Order $order): void
    {
        $this->em->persist($order);
        $this->em->flush();
    }

    public function find(OrderId $id): ?Order
    {
        return $this->em->find(Order::class, $id);
    }
}

final class StripePaymentGateway implements PaymentGateway
{
    public function __construct(private readonly \Stripe\StripeClient $stripe) {}

    public function charge(Money $amount, PaymentMethod $method): PaymentResult
    {
        // Stripe-specific logic isolated here
    }
}

// Composition root — the only place that knows concrete types
final class OrderServiceProvider
{
    public function register(Container $container): void
    {
        $container->bind(OrderRepository::class, DoctrineOrderRepository::class);
        $container->bind(PaymentGateway::class, StripePaymentGateway::class);
    }
}

// Test: swap infrastructure with fast in-memory doubles
final class OrderServiceTest extends \PHPUnit\Framework\TestCase
{
    public function testCreateOrderChargesAndSaves(): void
    {
        $repository = new InMemoryOrderRepository();
        $payment    = $this->createMock(PaymentGateway::class);
        $notifier   = new NullOrderNotifier();

        $payment->expects(self::once())
            ->method('charge')
            ->willReturn(PaymentResult::success('txn_123'));

        $service = new OrderService($repository, $payment, $notifier);
        $order   = $service->createOrder(CreateOrderCommand::fixture());

        self::assertTrue($order->isPaid());
        self::assertNotNull($repository->find($order->getId()));
    }
}
```

## Exceptions / trade-offs
Not every class needs an interface. A `Money` value object, a `DateRange`, or a utility like `Uuid::generate()` does not need an abstraction — there is no meaningful variation and no testing benefit. Create interfaces at architectural boundaries: between the application layer and infrastructure, between your code and third-party SDKs. Avoid the ritual of `FooInterface + Foo` pairs where only one implementation will ever exist.

## Static-analysis notes
PHPStan/Psalm enforce that constructor parameters match declared types, so they will catch accidental concrete dependencies when interfaces are declared. Deptrac can enforce architectural layer rules (e.g., "domain layer must not import from infrastructure layer") at the boundary level.

## Related topics
- [solid-ocp.md](solid-ocp.md) — strategy pattern implementations depend on the same interface DIP defines
- [solid-isp.md](solid-isp.md) — narrow interfaces make DIP more precise and mocking simpler
