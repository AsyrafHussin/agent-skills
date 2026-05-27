---
title: Nullable Types
impact: CRITICAL
impactDescription: Strong type safety across all PHP code
tags: php, types, strict-types, type-safety
---

# Nullable Types

## Why it matters
Using `null` as a default "I don't have one yet" placeholder corrupts domain meaning. When null appears in a return type or parameter that has not been explicitly declared nullable, callers have no contract to reason against — they either crash on a null dereference or defensively sprinkle `!== null` checks everywhere, both of which indicate a type-system failure upstream.

## Rule
Declare a type nullable (`?Type` or `Type|null`) only when null represents a real, intentional domain state. Never use `null` as a lazy alternative to a meaningful default or a thrown exception.

## Bad

```php
<?php

declare(strict_types=1);

class UserService
{
    // return type absent — callers can't tell null is possible
    public function findByEmail(string $email)
    {
        $data = $this->repository->findByEmail($email);
        if (!$data) {
            return null; // surprise null
        }
        return new User($data);
    }

    // deprecated implicit-nullable pattern
    public function setMiddleName(string $name = null): void
    {
        $this->middleName = $name;
    }

    // type says Subscription but actually returns null
    public function getActiveSubscription(): Subscription
    {
        return $this->subscriptions->getActive(); // may be null — lie in the contract
    }
}
```

## Better

```php
<?php

declare(strict_types=1);

class UserService
{
    public function findByEmail(string $email): ?User
    {
        $data = $this->repository->findByEmail($email);

        return $data !== null ? new User($data) : null;
    }

    public function setMiddleName(?string $name): void
    {
        $this->middleName = $name;
    }

    public function getActiveSubscription(): ?Subscription
    {
        return $this->subscriptions->getActive();
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\Exceptions\NoActiveSubscriptionException;
use App\Models\Subscription;
use App\Models\User;

final class UserService
{
    public function __construct(
        private readonly UserRepository $users,
        private readonly SubscriptionRepository $subscriptions,
    ) {}

    // null is a real domain state: "user with this email does not exist"
    public function findByEmail(string $email): ?User
    {
        return $this->users->findByEmail($email);
    }

    // null means "user has no middle name" — explicit optional
    public function setMiddleName(?string $name): void
    {
        $this->users->updateMiddleName($this->id, $name);
    }

    // caller decides whether null is acceptable
    public function activeSubscription(): ?Subscription
    {
        return $this->subscriptions->activeFor($this->id);
    }

    // use a throwing variant when null is not acceptable in context
    public function activeSubscriptionOrFail(): Subscription
    {
        $sub = $this->activeSubscription();

        if ($sub === null) {
            throw new NoActiveSubscriptionException($this->id);
        }

        return $sub;
    }
}

// call site
$user = $service->findByEmail($email);
$name = $user?->displayName() ?? 'Guest';
```

## Exceptions / trade-offs
- **Lazy initialisation**: When a property is null before first access (e.g., a cached value), nullable is correct — but consider using a sentinel object or lazy proxy instead to avoid null checks at every access point.
- **`?Type` vs `Type|null`**: Prefer `?Type` for single nullable types (shorter, clearer intent). Use `Type|null` only inside a multi-type union (e.g., `string|int|null`).
- **Implicit nullable is deprecated**: `function foo(string $x = null)` triggers a deprecation notice in PHP 8.4 — always write `?string $x = null` explicitly.

## Static-analysis notes
PHPStan and Psalm propagate nullability through the call graph and flag every unchecked nullable dereference. Use `->` only after a null-guard; use `?->` (nullsafe operator) for chained access. Both tools understand `?Type`, `Type|null`, and nullsafe chains.

## Version notes
`PHP 7.1+` — nullable types (`?Type`). `PHP 8.0+` — nullsafe operator (`?->`). `PHP 8.4` — implicit nullable parameter syntax is deprecated.

## Related topics
- [type-strict-mode.md](type-strict-mode.md) — strict mode makes null mismatches throw at runtime
- [type-return-types.md](type-return-types.md) — declare nullable return types explicitly
- [type-property-types.md](type-property-types.md) — nullable properties require explicit `?Type`
- [type-union-types.md](type-union-types.md) — `Type|null` in multi-type unions
