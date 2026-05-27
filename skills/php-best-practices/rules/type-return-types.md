---
title: Return Type Declarations
impact: CRITICAL
impactDescription: Strong type safety across all PHP code
tags: php, types, strict-types, type-safety
---

# Return Type Declarations

## Why it matters
A method without a declared return type is a black box: callers must read the implementation to know what comes back, assumptions diverge over time, and static analysis cannot verify downstream usage. When the return type is wrong or changes silently, bugs propagate through every caller before anyone notices.

## Rule
Every method and function must declare its return type. Use `void` for side-effect-only methods, `never` for methods that always throw or exit, and PHPDoc only when native PHP cannot express the shape (e.g., generic collections).

## Bad

```php
<?php

declare(strict_types=1);

class UserRepository
{
    // returns raw DB row array? a User object? null? nobody knows
    public function find(int $id)
    {
        return $this->db->query("SELECT * FROM users WHERE id = ?", [$id]);
    }

    // could return anything
    public function getActiveUsers()
    {
        return $this->db->query("SELECT * FROM users WHERE active = 1");
    }
}
```

## Better

```php
<?php

declare(strict_types=1);

class UserRepository
{
    public function find(int $id): ?User
    {
        $data = $this->db->query("SELECT * FROM users WHERE id = ?", [$id]);

        return $data !== null ? new User($data) : null;
    }

    /** @return User[] */
    public function getActiveUsers(): array
    {
        $results = $this->db->query("SELECT * FROM users WHERE active = 1");

        return array_map(static fn(array $row) => new User($row), $results);
    }

    public function save(User $user): void
    {
        $this->db->execute("INSERT INTO users ...", $user->toArray());
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

namespace App\Repositories;

use App\Models\User;
use App\Exceptions\UserNotFoundException;

final class UserRepository
{
    public function __construct(
        private readonly DatabaseConnection $db,
    ) {}

    public function find(int $id): ?User
    {
        $row = $this->db->selectOne(
            'SELECT * FROM users WHERE id = ?',
            [$id],
        );

        return $row !== null ? User::fromRow($row) : null;
    }

    public function findOrFail(int $id): User
    {
        $user = $this->find($id);

        if ($user === null) {
            throw new UserNotFoundException($id);
        }

        return $user;
    }

    /**
     * @return list<User>
     */
    public function findActive(): array
    {
        return array_map(
            static fn(array $row) => User::fromRow($row),
            $this->db->select('SELECT * FROM users WHERE active = 1'),
        );
    }

    public function delete(int $id): bool
    {
        return $this->db->execute('DELETE FROM users WHERE id = ?', [$id]) > 0;
    }

    public function save(User $user): void
    {
        $this->db->execute('INSERT INTO users ...', $user->toArray());
    }
}
```

## Exceptions / trade-offs
- **`static` return type**: Use `static` (not `self`) for fluent builder methods that must be overridable by subclasses.
- **Generic collections**: Native PHP cannot express `list<User>`; add PHPDoc `@return list<User>` alongside the native `array` declaration for static-analysis tools.
- **Covariant returns**: Child classes may narrow a return type (e.g., `?User` → `User`) but cannot widen it. Design interfaces with the most general type the caller needs.

## Static-analysis notes
PHPStan (level 5+) and Psalm flag missing return types and return-type mismatches. The `@return` PHPDoc for generic arrays is verified by both tools. PhpStorm infers return types from declarations for autocompletion and safe-delete refactoring.

## Version notes
`PHP 7.0+` — scalar return types. `PHP 7.1+` — `void`. `PHP 8.1+` — `never`.

## Related topics
- [type-strict-mode.md](type-strict-mode.md) — strict mode enforces return types at runtime
- [type-parameter-types.md](type-parameter-types.md) — pair return types with typed parameters
- [type-void-never.md](type-void-never.md) — when a method returns nothing or never returns
- [type-nullable-types.md](type-nullable-types.md) — when null is a valid return value
