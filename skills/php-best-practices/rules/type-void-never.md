---
title: Void and Never Return Types
impact: CRITICAL
impactDescription: Strong type safety across all PHP code
tags: php, types, strict-types, type-safety
---

# Void and Never Return Types

## Why it matters
`void` and `never` are not stylistic choices — they carry hard semantic guarantees. A `void` method that accidentally returns a value will trigger a `TypeError`. A `never` method that returns normally will too. More importantly, static-analysis tools use `never` to determine reachability: code after a `never` call is dead code, which unlocks better type narrowing and eliminates false-positive "variable may be undefined" warnings.

## Rule
Use `void` when a method performs a side effect and returns no value. Use `never` when a method always exits the current execution path — by throwing an exception, calling `exit`, or looping forever. Never use `void` as a synonym for `never`.

## Bad

```php
<?php

declare(strict_types=1);

class EventDispatcher
{
    // missing return type — unclear if something is returned
    public function dispatch(Event $event)
    {
        foreach ($this->listeners as $listener) {
            $listener->handle($event);
        }
    }
}

class ExceptionHandler
{
    // void is wrong here — method never returns
    public function handleFatal(\Throwable $e): void
    {
        $this->logger->critical($e->getMessage());
        exit(1); // exit is reachable — static analysis is confused
    }

    // also wrong — always throws, but declared void
    public function abort(int $code): void
    {
        throw new \RuntimeException("HTTP {$code}");
    }
}
```

## Better

```php
<?php

declare(strict_types=1);

class EventDispatcher
{
    public function dispatch(Event $event): void
    {
        foreach ($this->listeners as $listener) {
            $listener->handle($event);
        }
    }
}

class ExceptionHandler
{
    public function handleFatal(\Throwable $e): never
    {
        $this->logger->critical($e->getMessage());
        exit(1);
    }

    public function abort(int $code): never
    {
        throw new \RuntimeException("HTTP {$code}");
    }
}
```

## Best

```php
<?php

declare(strict_types=1);

namespace App\Events;

use App\Exceptions\HttpException;
use Psr\Log\LoggerInterface;

final class EventDispatcher
{
    /** @var array<string, list<callable>> */
    private array $listeners = [];

    public function __construct(
        private readonly LoggerInterface $logger,
    ) {}

    public function listen(string $event, callable $listener): void
    {
        $this->listeners[$event][] = $listener;
    }

    public function dispatch(object $event): void
    {
        $name = $event::class;

        foreach ($this->listeners[$name] ?? [] as $listener) {
            $listener($event);
        }
    }
}

final class ErrorHandler
{
    public function __construct(
        private readonly LoggerInterface $logger,
    ) {}

    public function handleFatal(\Throwable $e): never
    {
        $this->logger->critical($e->getMessage(), ['exception' => $e]);
        http_response_code(500);
        exit(1);
    }

    public function abort(int $status, string $message = ''): never
    {
        throw new HttpException($status, $message);
    }
}

// never enables type narrowing — static analysis knows $value is int after this
function requireInt(mixed $value): int
{
    if (! is_int($value)) {
        throw new \TypeError('Expected int, got ' . get_debug_type($value));
    }

    return $value; // tools know this line is reachable and $value is int
}
```

## Exceptions / trade-offs
- **`never` and interfaces**: An interface method can declare `void` while an implementation returns `never` — `never` is a subtype of every type including `void`. The reverse is not allowed.
- **Daemon loops**: Methods containing `while (true) { ... }` with no reachable `break` are legitimately `never`. Annotate them as such so callers know a call to the method does not return.
- **`exit` in tests**: Avoid `never` methods that call `exit` directly in code under test; inject a termination strategy so the test can substitute a throwing implementation.

## Static-analysis notes
PHPStan and Psalm use `never` for dead-code detection and type narrowing. Any code after a call to a `never` method is flagged as unreachable. Both tools verify that a `never` function actually throws or exits on all code paths. Psalm's `assert-never` annotation composes well with `never` return types in exhaustiveness checks.

## Version notes
`PHP 7.1+` — `void`. `PHP 8.1+` — `never`.

## Related topics
- [type-return-types.md](type-return-types.md) — declaring return types in general
- [type-strict-mode.md](type-strict-mode.md) — strict mode enforces void/never contracts
- [type-union-types.md](type-union-types.md) — never as a member of union types in advanced patterns
