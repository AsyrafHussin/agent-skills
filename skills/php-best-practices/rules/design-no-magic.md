---
title: Avoid Magic-Heavy Design
impact: HIGH
impactDescription: PHP design patterns and value object discipline
tags: php, design-patterns, value-objects
---

# Avoid Magic-Heavy Design

## Why it matters

`__get`, `__set`, `__call`, and `__callStatic` make all property access and method dispatch invisible to static analysis. PHPStan returns `mixed` for every `__get` read and cannot verify that `$user->getNmae()` is a typo until it blows up at runtime. IDE autocompletion stops working. Refactoring becomes guesswork because there is no tooling to find all call sites of a dynamically dispatched method.

## Rule

Prefer explicit, typed properties and methods. Use magic methods only in framework infrastructure (ORM base classes, proxies, dynamic builders) where the trade-off is justified and PHPDoc stubs compensate for the lost static visibility.

## Bad

```php
<?php

declare(strict_types=1);

// __get/__set/__call — everything hidden from static analysis
class User
{
    /** @var array<string, mixed> */
    private array $attributes = [];

    public function __get(string $name): mixed
    {
        return $this->attributes[$name] ?? null;
    }

    public function __set(string $name, mixed $value): void
    {
        $this->attributes[$name] = $value;
    }

    public function __call(string $name, array $args): mixed
    {
        if (str_starts_with($name, 'get')) {
            return $this->attributes[lcfirst(substr($name, 3))] ?? null;
        }
        throw new \BadMethodCallException("Method {$name} not found");
    }
}

$user = new User();
$user->email = 'alice@example.com'; // __set — no type check
$user->name  = 42;                  // accepted silently — wrong type
$user->getNmae();                   // typo — silently returns null
echo $user->email;                  // __get — PHPStan infers mixed
```

## Better

```php
<?php

declare(strict_types=1);

// Explicit properties — typed, but still mutable after construction
class User
{
    public string $id;
    public string $email;
    public string $name;

    public function __construct(string $id, string $email, string $name)
    {
        $this->id    = $id;
        $this->email = $email;
        $this->name  = $name;
    }

    public function isActive(): bool
    {
        return true;
    }
}

// PHPStan and IDEs can now see all properties and methods.
// Typos on property names produce static-analysis errors.
$user = new User('1', 'alice@example.com', 'Alice');
echo $user->email;
```

## Best

```php
<?php

declare(strict_types=1);

// Explicit, typed with value objects, readonly-by-default — no magic anywhere
final class User
{
    public function __construct(
        public readonly UserId $id,
        private Email $email,
        private UserName $name,
        private UserStatus $status = UserStatus::Active,
    ) {}

    public function getEmail(): Email
    {
        return $this->email;
    }

    public function getName(): UserName
    {
        return $this->name;
    }

    public function isActive(): bool
    {
        return $this->status === UserStatus::Active;
    }

    public function deactivate(): void
    {
        $this->status = UserStatus::Inactive;
    }

    // No __get, no __set, no __call
}

// PHPStan sees every property and method. Value objects enforce invariants.
// IDEs autocomplete getEmail() and know it returns Email, not mixed.
$user = new User(new UserId(1), new Email('alice@example.com'), new UserName('Alice'));
echo $user->getEmail()->value;  // string — not mixed
```

## When magic is acceptable

Framework infrastructure — ORM base classes, dynamic proxies, and autowiring containers — legitimately requires magic. Contain it and compensate with PHPDoc stubs:

```php
<?php

declare(strict_types=1);

/**
 * Eloquent-style base model — magic is intentional at this layer.
 * PHPStan plugin (larastan/larastan) provides type stubs.
 *
 * @method static static where(string $column, mixed $value)
 * @method static list<static> get()
 * @property-read int $id
 * @property string $email
 */
abstract class Model
{
    /** @param list<mixed> $args */
    public function __call(string $name, array $args): mixed
    {
        return $this->newQuery()->$name(...$args);
    }

    /** @param list<mixed> $args */
    public static function __callStatic(string $name, array $args): mixed
    {
        return (new static())->$name(...$args);
    }
}

// Explicit subclass restores full type safety for domain code:
final class UserModel extends Model
{
    public static function active(): static
    {
        return static::where('status', 'active');
    }
}
```

## Exceptions / trade-offs

- **ORM base classes** (Eloquent, Doctrine proxies): magic is the intended API. Use PHPDoc `@method` stubs or a PHPStan plugin to restore analysis coverage.
- **Dynamic proxies and aspect wrappers**: justified if properly stubbed.
- **`__invoke`**: acceptable and analyzable — it makes a class a first-class callable with a known signature.
- `__toString` and `__clone` are fine — they have defined, predictable semantics.

Do not use magic merely to avoid typing out getter methods.

## Static-analysis notes

- `__get` returns `mixed` unless annotated with a `@property` stub in the class docblock.
- PHPStan `@method` stubs in the docblock restore analysis for `__call` dispatchers.
- Explicit typed properties and methods give PHPStan / Psalm / IDEs full visibility with zero annotation overhead.
- The PHPStan `phpstan/phpstan-strict-rules` package flags undocumented magic usage.

## Related topics

- [design-value-objects.md](design-value-objects.md) — replace primitive attributes bags with typed value objects
- [type-property-types.md](type-property-types.md) — type every property explicitly
- [type-mixed-avoid.md](type-mixed-avoid.md) — avoid `mixed` returns that magic methods produce
- [phpstan-phpdoc.md](phpstan-phpdoc.md) — use PHPDoc stubs where magic cannot be avoided
