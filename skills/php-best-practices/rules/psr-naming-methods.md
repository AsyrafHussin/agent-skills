---
title: PSR Method Naming
impact: HIGH
impactDescription: PSR standards and code structure conventions
tags: php, psr, coding-standards
---

# PSR Method Naming

## Why it matters
`handle()`, `process()`, `execute()`, and `doStuff()` are non-names. They force the reader into the method body just to understand what the method does. A well-named method is a unit of self-documentation: `validateBillingAddress()`, `markAsPaid()`, `findByEmail()` are searchable, greppable, and communicate intent without a comment. Generic names also hide design problems — a method called `handle()` is usually doing too many things.

## Rule
Method names reveal action or query intent. Use `findBy*()` for queries that may return null, `getBy*()` for collections, `markAs*()` for state transitions, `validateX()` for validation. Use `is*/has*/can*` prefixes for booleans. Avoid vague names like `handle()`, `process()`, `execute()`, `manage()`.

## Bad

```php
<?php

declare(strict_types=1);

final class UserService
{
    public function get_user_by_id(int $id): ?User {} // underscores
    public function FindActive(): array {}             // uppercase first letter
    public function user(int $id): User {}             // no verb — is this a getter? factory? query?
    public function getUsrByEml(string $email): ?User {} // abbreviations
    public function checkActive(): bool {}             // 'check' is vague; use 'is'
    public function process(): void {}                 // process what?
    public function handle(): void {}                  // handle what?
    public function doStuff(): void {}                 // literally nothing
}
```

## Better

```php
<?php

declare(strict_types=1);

final class UserService
{
    public function findById(UserId $id): ?User
    {
        return $this->repository->find($id);
    }

    public function findByEmail(Email $email): ?User
    {
        return $this->repository->findByEmail($email);
    }

    public function isActive(User $user): bool
    {
        return $user->status === UserStatus::Active;
    }

    public function activate(User $user): void
    {
        $user->markAsActive();
        $this->repository->save($user);
    }

    public function deactivate(User $user): void
    {
        $user->markAsInactive();
        $this->repository->save($user);
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

final class UserService
{
    // Queries returning single nullable result: findBy*
    public function findByEmail(Email $email): ?User
    {
        return $this->repository->findByEmail($email);
    }

    // Queries that must exist: getBy* or findOrFail
    public function findOrFail(UserId $id): User
    {
        return $this->repository->find($id)
            ?? throw new UserNotFoundException($id);
    }

    // Boolean queries: is*/has*/can*
    public function isEmailTaken(Email $email): bool
    {
        return $this->repository->findByEmail($email) !== null;
    }

    public function hasVerifiedEmail(User $user): bool
    {
        return $user->emailVerifiedAt() !== null;
    }

    public function canResetPassword(User $user): bool
    {
        return $user->isActive() && $this->hasVerifiedEmail($user);
    }

    // State transitions: markAs* or domain verb
    public function markEmailAsVerified(User $user): void
    {
        $user->verifyEmail(new \DateTimeImmutable());
        $this->repository->save($user);
    }

    public function suspendAccount(User $user, string $reason): void
    {
        $user->suspend($reason);
        $this->repository->save($user);
        $this->notifier->notifySuspension($user);
    }

    // Commands: specific verb + noun
    public function assignRole(User $user, Role $role): void
    {
        $user->addRole($role);
        $this->repository->save($user);
    }

    public function revokeRole(User $user, Role $role): void
    {
        $user->removeRole($role);
        $this->repository->save($user);
    }

    // Calculations: calculate* or compute*
    public function calculateDaysSinceRegistration(User $user): int
    {
        return $user->registeredAt()->diff(new \DateTimeImmutable())->days;
    }
}

// Entity methods follow the same convention
final class User
{
    // State transitions as domain verbs
    public function verifyEmail(\DateTimeImmutable $at): void
    {
        $this->emailVerifiedAt = $at;
    }

    public function suspend(string $reason): void
    {
        $this->status = UserStatus::Suspended;
        $this->suspensionReason = $reason;
    }

    // Boolean queries on the entity itself
    public function isActive(): bool
    {
        return $this->status === UserStatus::Active;
    }

    public function isSuspended(): bool
    {
        return $this->status === UserStatus::Suspended;
    }

    public function hasRole(Role $role): bool
    {
        return in_array($role, $this->roles, true);
    }
}
```

## Exceptions / trade-offs
Framework lifecycle hooks (`handle()` on a Job, `boot()` on a ServiceProvider, `up()` on a Migration) are mandated by the framework — do not rename them. Event listeners named `handle()` are a Laravel convention. Magic methods (`__construct`, `__toString`, `__invoke`) follow PHP convention. The rule applies to domain and application code you control.

## Static-analysis notes
PHPStan and Psalm do not enforce naming conventions. PHP_CodeSniffer with a naming sniff, or `phparkitect`, can lint method name patterns (e.g., "methods returning `bool` must start with `is`, `has`, or `can`"). Psalm's `@return bool` assertion checks combined with naming conventions create a self-documenting signal.

## Related topics
- [psr-naming-classes.md](psr-naming-classes.md) — class names set the context that method names fill
- [solid-srp.md](solid-srp.md) — vague method names often indicate a method is doing too many things
