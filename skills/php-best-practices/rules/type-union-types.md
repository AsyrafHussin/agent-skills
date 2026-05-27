---
title: Union Types
impact: CRITICAL
impactDescription: Strong type safety across all PHP code
tags: php, types, strict-types, type-safety
---

# Union Types

## Why it matters
Union types used carelessly become a machine-readable version of "I didn't think this through." When a parameter genuinely accepts `string|int` for a domain reason (e.g., a value that arrives from both user input and a DB integer column), the union is correct. When it is used because the developer wasn't sure which type to commit to, it is lazy domain modeling that hides design debt and makes every caller bear the branching cost.

## Rule
Use union types only when each member of the union represents a real, separately meaningful alternative state. Prefer purpose-built value objects over wide unions. Reach for `mixed` last — and see [type-mixed-avoid.md](type-mixed-avoid.md).

## Bad

```php
<?php

declare(strict_types=1);

class ConfigLoader
{
    /**
     * @param string|array $source
     * @return mixed
     */
    public function load($source) // no native type — docblock does no enforcement
    {
        if (is_string($source)) {
            return $this->loadFromFile($source);
        }
        return $this->loadFromArray($source);
    }

    // mixed is not a union — it disables all checking
    public function getValue(string $key): mixed
    {
        return $this->config[$key] ?? null;
    }
}
```

## Better

```php
<?php

declare(strict_types=1);

class ConfigLoader
{
    /** @param string|array<string, mixed> $source */
    public function load(string|array $source): array
    {
        if (is_string($source)) {
            return $this->loadFromFile($source);
        }
        return $this->loadFromArray($source);
    }

    public function getValue(string $key): string|int|bool|null
    {
        return $this->config[$key] ?? null;
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

namespace App\Config;

final class ConfigLoader
{
    /** @var array<string, string|int|bool|null> */
    private array $data = [];

    /**
     * Accepts a file path or a pre-parsed config array.
     * Both are real entry points in the application bootstrap.
     *
     * @param array<string, string|int|bool|null> $defaults
     */
    public function load(string|\SplFileInfo $source, array $defaults = []): static
    {
        $parsed = $source instanceof \SplFileInfo
            ? $this->parseFile($source)
            : $this->parseFile(new \SplFileInfo($source));

        $this->data = array_merge($defaults, $parsed);

        return $this;
    }

    public function get(string $key): string|int|bool|null
    {
        return $this->data[$key] ?? null;
    }

    public function set(string $key, string|int|bool $value): void
    {
        $this->data[$key] = $value;
    }

    private function parseFile(\SplFileInfo $file): array
    {
        return parse_ini_file($file->getRealPath(), true) ?: [];
    }
}

// Logger that genuinely handles two message shapes
final class StructuredLogger
{
    public function log(string|Stringable $message, string $level = 'info'): void
    {
        $this->write($level, (string) $message);
    }

    private function write(string $level, string $text): void
    {
        // ...
    }
}
```

## Exceptions / trade-offs
- **`int|string` identifiers**: Common for IDs that arrive as strings from HTTP but are stored as integers — a legitimate union if you handle both branches explicitly.
- **PHP built-in flexibility**: Some PHP built-ins return `int|false` (e.g., `strpos`). Mirror that signature only at API boundary adapters; normalise to a typed value object inside your domain.
- **Avoid `T|null` sprawl**: When more than two members of a union can be null, consider whether a value object or a `Result<T>` type would be cleaner.

## Static-analysis notes
PHPStan and Psalm require exhaustive type narrowing inside the function body when a union is used. Both tools will flag unreachable branches and missing narrowing. Use `is_string()`, `is_int()`, `instanceof`, or `match` to narrow unions — tools verify coverage.

## Version notes
`PHP 8.0+` — native union types. PHPDoc `@param string|array` worked before 8.0 but was unenforced at runtime.

## Related topics
- [type-strict-mode.md](type-strict-mode.md) — strict mode enforces union members at runtime
- [type-nullable-types.md](type-nullable-types.md) — `Type|null` as the simplest union
- [type-intersection-types.md](type-intersection-types.md) — when a value must satisfy multiple contracts
- [type-mixed-avoid.md](type-mixed-avoid.md) — what to use instead of mixed
