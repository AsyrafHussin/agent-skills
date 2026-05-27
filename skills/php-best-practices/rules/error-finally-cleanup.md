---
title: Use `finally` for Guaranteed Cleanup, Not Control Flow
impact: HIGH
impactDescription: Robust error handling and exception design
tags: php, error-handling, exceptions
---

# Use `finally` for Guaranteed Cleanup, Not Control Flow

## Why it matters
Resources left open after an exception — file handles, database locks, connections, temporary state — cause correctness bugs and resource exhaustion that are difficult to reproduce. Duplicating cleanup in both `try` and `catch` creates drift: one copy gets updated, the other does not.

## Rule
Use `finally` to guarantee cleanup runs regardless of the execution path. Do not use it for branching, returning calculated values, or normal business logic — that makes control flow hard to follow.

## Bad
```php
<?php

declare(strict_types=1);

// Cleanup duplicated; skipped entirely if an unexpected exception occurs
function processFile(string $path): array
{
    $handle = fopen($path, 'r');

    try {
        $data = parseContents($handle);
        fclose($handle); // skipped if parseContents throws
        return $data;
    } catch (ParseException $e) {
        fclose($handle); // duplicated
        throw $e;
        // any other exception leaks the handle
    }
}
```

## Better
```php
<?php

declare(strict_types=1);

// Finally added, but catch block re-closes unnecessarily
function processFile(string $path): array
{
    $handle = fopen($path, 'r');

    try {
        return parseContents($handle);
    } catch (ParseException $e) {
        fclose($handle); // redundant — finally will run anyway
        throw $e;
    } finally {
        fclose($handle);
    }
}
```

## Best
```php
<?php

declare(strict_types=1);

// File handle: finally closes regardless of success or any exception
function processFile(string $path): array
{
    $handle = fopen($path, 'r');

    try {
        return parseContents($handle);
    } finally {
        fclose($handle);
    }
}

// Distributed lock: always released
function updateInventory(PDO $pdo, int $productId, int $delta): void
{
    $lock = Cache::lock("product:{$productId}", ttl: 10);
    $lock->acquire();

    try {
        $stmt = $pdo->prepare('UPDATE products SET stock = stock + ? WHERE id = ?');
        $stmt->execute([$delta, $productId]);
    } finally {
        $lock->release();
    }
}

// Locale: always restored after the callback
function withLocale(string $locale, callable $callback): mixed
{
    $original = setlocale(LC_ALL, '0');

    try {
        setlocale(LC_ALL, $locale);
        return $callback();
    } finally {
        setlocale(LC_ALL, $original);
    }
}

// Transaction: explicit rollback on failure, then rethrow
function transferFunds(PDO $pdo, int $fromId, int $toId, float $amount): void
{
    $pdo->beginTransaction();

    try {
        debit($pdo, $fromId, $amount);
        credit($pdo, $toId, $amount);
        $pdo->commit();
    } catch (\Throwable $e) {
        $pdo->rollBack();
        throw $e;
    }
}
```

## Exceptions / trade-offs
`finally` runs even when the `try` block calls `return`, which is usually correct for cleanup. Avoid putting a `return` statement inside `finally` itself — it silently discards exceptions and overrides the `try` return value.

## Static-analysis notes
PHPStan can flag unreachable code after `return` inside `try` when `finally` also returns. Neither PHPStan nor Psalm currently warn about `return` inside `finally`, so enforce that via code review.

## Related topics
- [error-try-catch-specific.md](error-try-catch-specific.md) — catching exceptions at the right scope
- [error-never-suppress.md](error-never-suppress.md) — other patterns that hide failures
