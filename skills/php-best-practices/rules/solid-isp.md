---
title: Interface Segregation Principle (ISP)
impact: HIGH
impactDescription: SOLID principles and clean OOP design
tags: php, solid, oop, design-patterns
---

# Interface Segregation Principle (ISP)

## Why it matters
A class forced to implement methods it does not use is a class that either lies (returns null / throws exceptions) or stays permanently broken. Every `throw new \LogicException('Not implemented')` in a concrete class is an ISP violation waiting to crash in production. Wide interfaces also make mocking painful in tests — you mock twelve methods when you care about two.

## Rule
Small, focused interfaces — no kitchen-sink contracts. If a class implementing an interface is forced to leave methods empty or throw exceptions, the interface is too wide. Split it.

## Bad

```php
<?php

declare(strict_types=1);

interface WorkerInterface
{
    public function work(): void;
    public function eat(): void;
    public function sleep(): void;
    public function attendMeeting(): void;
    public function submitTimesheet(): void;
    public function requestVacation(): void;
}

// Robot must lie about four methods it will never legitimately implement
final class Robot implements WorkerInterface
{
    public function work(): void { /* actual logic */ }
    public function eat(): void { throw new \LogicException("Robots don't eat"); }
    public function sleep(): void { throw new \LogicException("Robots don't sleep"); }
    public function attendMeeting(): void { throw new \LogicException("Robots don't attend meetings"); }
    public function submitTimesheet(): void { throw new \LogicException("Robots don't submit timesheets"); }
    public function requestVacation(): void { throw new \LogicException("Robots don't take vacation"); }
}
```

## Better

```php
<?php

declare(strict_types=1);

interface Workable
{
    public function work(): void;
}

interface HumanWorkerContract extends Workable
{
    public function eat(): void;
    public function sleep(): void;
    public function attendMeeting(): void;
    public function submitTimesheet(): void;
    public function requestVacation(): void;
}

// Robot only implements what it actually does
final class Robot implements Workable
{
    public function work(): void { /* actual logic */ }
}

final class Employee implements HumanWorkerContract
{
    public function work(): void { /* work logic */ }
    public function eat(): void { /* lunch break */ }
    public function sleep(): void { /* after hours */ }
    public function attendMeeting(): void { /* calendar invite */ }
    public function submitTimesheet(): void { /* HR system */ }
    public function requestVacation(): void { /* PTO request */ }
}
```

## Best

```php
<?php

declare(strict_types=1);

// Repositories segregated by capability, not by CRUD completeness

interface FindsUsers
{
    public function find(UserId $id): ?User;
    public function findByEmail(Email $email): ?User;
}

interface SavesUsers
{
    public function save(User $user): void;
}

interface PaginatesUsers
{
    public function paginate(int $page, int $perPage): PaginatedResult;
}

interface SearchesUsers
{
    /** @return list<User> */
    public function search(string $query): array;
}

// Full-featured repository for normal use cases
final class UserRepository implements FindsUsers, SavesUsers, PaginatesUsers, SearchesUsers
{
    public function __construct(private readonly \PDO $pdo) {}

    public function find(UserId $id): ?User { /* ... */ }
    public function findByEmail(Email $email): ?User { /* ... */ }
    public function save(User $user): void { /* ... */ }
    public function paginate(int $page, int $perPage): PaginatedResult { /* ... */ }
    public function search(string $query): array { /* ... */ }
}

// Read-only audit log repository: no save, no search
final class AuditLogUserRepository implements FindsUsers, PaginatesUsers
{
    public function __construct(private readonly \PDO $pdo) {}

    public function find(UserId $id): ?User { /* ... */ }
    public function paginate(int $page, int $perPage): PaginatedResult { /* ... */ }
}

// Service declares exactly the dependency slice it needs
final class UserListService
{
    public function __construct(
        private readonly FindsUsers&PaginatesUsers $repository,
    ) {}

    public function getPage(int $page): PaginatedResult
    {
        return $this->repository->paginate($page, 20);
    }
}
```

## Exceptions / trade-offs
Intersection types (`FindsUsers&PaginatesUsers`) require PHP 8.1+. In older codebases, a combined interface (`ReadableUserRepository extends FindsUsers, PaginatesUsers`) is a reasonable bridge. Do not over-segregate: one interface per method is not the goal. Group by cohesive use cases (read, write, search) rather than by individual method.

## Static-analysis notes
PHPStan and Psalm will error on missing interface method implementations. Psalm's `@psalm-require-implements` can enforce which interface a class must implement. Neither tool will automatically detect that an implementation is throwing `LogicException` for unused methods — that requires code review.

## Related topics
- [solid-lsp.md](solid-lsp.md) — forced `throw new LogicException` in implementations is an LSP violation too
- [solid-dip.md](solid-dip.md) — narrower interfaces make dependency injection more precise
