---
title: Single Responsibility Principle (SRP)
impact: HIGH
impactDescription: SOLID principles and clean OOP design
tags: php, solid, oop, design-patterns
---

# Single Responsibility Principle (SRP)

## Why it matters
A class that does payment processing, sends emails, and logs to files has three reasons to change: business rules around payment, notification policy, and logging infrastructure. Each of those changes touches the same class, making every change a potential regression in the other two concerns, and making each concern impossible to test in isolation.

## Rule
Give a class one coherent purpose, not one method. "One reason to change" means one stakeholder or one domain concern drives all mutations to that class — not that the class has exactly one method.

## Bad

```php
<?php

declare(strict_types=1);

final class User
{
    public function __construct(
        private int $id,
        private string $name,
        private string $email,
    ) {}

    public function save(): void
    {
        $pdo = new PDO('mysql:host=localhost;dbname=app', 'user', 'pass');
        $stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (?, ?)');
        $stmt->execute([$this->name, $this->email]);
    }

    public function sendWelcomeEmail(): void
    {
        mail($this->email, 'Welcome!', "Hello {$this->name}");
    }

    public function toJson(): string
    {
        return json_encode(['id' => $this->id, 'name' => $this->name], JSON_THROW_ON_ERROR);
    }

    public function validate(): array
    {
        $errors = [];
        if (empty($this->name)) {
            $errors[] = 'Name is required';
        }
        if (!filter_var($this->email, FILTER_VALIDATE_EMAIL)) {
            $errors[] = 'Invalid email';
        }
        return $errors;
    }
}
```

## Better

```php
<?php

declare(strict_types=1);

// Domain entity — only business rules
final class User
{
    public function __construct(
        private readonly UserId $id,
        private readonly Email $email,
        private readonly UserName $name,
        private UserStatus $status = UserStatus::Active,
    ) {}

    public function getId(): UserId { return $this->id; }
    public function getEmail(): Email { return $this->email; }
    public function getName(): UserName { return $this->name; }
    public function isActive(): bool { return $this->status === UserStatus::Active; }
    public function activate(): void { $this->status = UserStatus::Active; }
    public function deactivate(): void { $this->status = UserStatus::Inactive; }
}

// Persistence — only how a User is stored
interface UserRepository
{
    public function find(UserId $id): ?User;
    public function save(User $user): void;
    public function delete(User $user): void;
}

final class DatabaseUserRepository implements UserRepository
{
    public function __construct(private readonly \PDO $pdo) {}

    public function find(UserId $id): ?User
    {
        $stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = ?');
        $stmt->execute([$id->value]);
        $data = $stmt->fetch(\PDO::FETCH_ASSOC);
        return $data ? $this->hydrate($data) : null;
    }

    public function save(User $user): void { /* save implementation */ }
    public function delete(User $user): void { /* delete implementation */ }

    private function hydrate(array $data): User { /* hydration logic */ }
}

// Notification — only how a User is welcomed
final class UserEmailNotifier
{
    public function __construct(
        private readonly MailerInterface $mailer,
        private readonly EmailTemplateRenderer $renderer,
    ) {}

    public function sendWelcomeEmail(User $user): void
    {
        $body = $this->renderer->render('welcome', ['name' => $user->getName()->value]);
        $this->mailer->send($user->getEmail()->value, 'Welcome!', $body);
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

// Domain entity: only business rules for what a User is
final class User
{
    public function __construct(
        private readonly UserId $id,
        private readonly Email $email,
        private readonly UserName $name,
        private UserStatus $status = UserStatus::Active,
    ) {}

    public function getId(): UserId { return $this->id; }
    public function getEmail(): Email { return $this->email; }
    public function getName(): UserName { return $this->name; }
    public function isActive(): bool { return $this->status === UserStatus::Active; }
    public function deactivate(): void { $this->status = UserStatus::Inactive; }
}

// Persistence: only how a User is stored
interface UserRepository
{
    public function find(UserId $id): ?User;
    public function save(User $user): void;
}

// Notification: only how a User is welcomed
final class UserWelcomeMailer
{
    public function __construct(
        private readonly MailerInterface $mailer,
        private readonly TemplateRenderer $renderer,
    ) {}

    public function send(User $user): void
    {
        $body = $this->renderer->render('welcome', ['name' => $user->getName()->value]);
        $this->mailer->send($user->getEmail()->value, 'Welcome!', $body);
    }
}

// Application service: thin orchestrator, no domain logic
final class RegisterUserHandler
{
    public function __construct(
        private readonly UserRepository $repository,
        private readonly UserWelcomeMailer $mailer,
        private readonly UserFactory $factory,
    ) {}

    public function handle(RegisterUserCommand $command): User
    {
        $user = $this->factory->fromCommand($command);
        $this->repository->save($user);
        $this->mailer->send($user);
        return $user;
    }
}
```

## Exceptions / trade-offs
Small value objects may legitimately handle both data and formatting (e.g., a `Money` class that also knows how to format itself). Don't split trivially small classes for the sake of the principle — the goal is cohesion, not class count. An anemic domain model split so far that the `User` class has no behavior is also a violation.

## Static-analysis notes
PHPStan and Psalm cannot enforce SRP directly. Watch for: classes with many constructor parameters (5+), methods with low cohesion (operate on disjoint sets of properties), and classes that import from radically different namespaces (e.g., domain + HTTP + database all in one class).

## Related topics
- [solid-ocp.md](solid-ocp.md) — how to extend instead of modify once responsibilities are separated
- [solid-dip.md](solid-dip.md) — depend on abstractions to decouple separated responsibilities
