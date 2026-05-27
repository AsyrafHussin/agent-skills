---
title: Property Hooks
impact: CRITICAL
impactDescription: Modern PHP 8.x language features
tags: php, modern-php, php8
---

# Property Hooks

## Why it matters
Separate getter and setter methods scatter access logic away from the property definition, making it harder to see what a property actually does at a glance. PHP 8.4 property hooks co-locate that logic directly with the property declaration, reducing boilerplate without hiding behaviour — as long as hooks stay pure.

## Rule
Use property hooks for focused, pure access logic (normalisation, formatting, computed values) co-located with the property. Never put side effects inside a hook.

## Bad
```php
<?php

declare(strict_types=1);

final class User
{
    private string $email;
    private string $firstName;
    private string $lastName;

    public function getEmail(): string        { return $this->email; }
    public function setEmail(string $email): void
    {
        $this->email = strtolower(trim($email));
    }

    public function getFirstName(): string    { return $this->firstName; }
    public function getLastName(): string     { return $this->lastName; }
    public function getFullName(): string     { return "{$this->firstName} {$this->lastName}"; }
}
```

## Better
```php
<?php

declare(strict_types=1);

final class User
{
    public string $email {
        set(string $value) {
            $this->email = strtolower(trim($value));
        }
    }

    public function __construct(
        public readonly string $firstName,
        public readonly string $lastName,
    ) {}
}

$user = new User('Alice', 'Smith');
$user->email = '  ALICE@EXAMPLE.COM  ';
echo $user->email;  // alice@example.com
```

## Best
```php
<?php

declare(strict_types=1);

final class User
{
    // set hook normalises input; get hook is implicit (returns stored value)
    public string $email {
        set(string $value) {
            $this->email = strtolower(trim($value));
        }
    }

    // Virtual computed property — no backing storage, no setter
    public string $fullName {
        get => "{$this->firstName} {$this->lastName}";
    }

    public function __construct(
        public readonly string $firstName,
        public readonly string $lastName,
    ) {}
}

$user = new User('Alice', 'Smith');
$user->email = '  ALICE@EXAMPLE.COM  ';

echo $user->email;    // alice@example.com
echo $user->fullName; // Alice Smith  (computed, no stored field)
```

## Exceptions / trade-offs
Do **not** put side effects inside hooks — no database queries, HTTP requests, event dispatching, or logging. Hooks fire on every property access, and hidden I/O in a property access is a debugging and performance trap. For side effects, use explicit methods. Property hooks also cannot be used on `readonly` properties.

## Static-analysis notes
PHPStan and Psalm 8.4+ understand property hooks, infer the type of virtual (get-only) properties, and report type mismatches in hook bodies. IDEs provide navigation directly into hook bodies and show hook presence in property tooltips.

## Version notes
`PHP 8.4+`

## Related topics
- [modern-asymmetric-visibility.md](modern-asymmetric-visibility.md) — for read-public / write-private without logic
- [modern-readonly-properties.md](modern-readonly-properties.md) — for fully immutable properties
