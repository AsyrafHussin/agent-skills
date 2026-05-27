---
title: Use Generators to Stream Large Datasets
impact: MEDIUM
impactDescription: Runtime performance and memory efficiency
tags: php, performance, optimization
---

# Use Generators to Stream Large Datasets

## Why it matters
Loading 100,000 database rows into a PHP array allocates memory proportional to the entire result set before processing a single row. A generator yields one item at a time, keeping peak memory constant regardless of dataset size. This matters for CSV exports, report generation, batch processing, and any operation where the data exceeds available RAM.

## Rule
Use `yield` to stream sequences that are large, unbounded, or expensive to materialize. A generator processes one item at a time and uses constant memory. Return typed `\Generator` from the function signature so callers and tools understand the contract.

## Bad
```php
<?php

declare(strict_types=1);

// Loads every row into memory before iteration begins
function getAllUsers(\PDO $pdo): array
{
    return $pdo->query('SELECT * FROM users')->fetchAll(); // 1 M rows = large spike
}

// Loads entire file
function readLines(string $path): array
{
    return file($path); // 1 GB file = 1 GB + memory
}

foreach (getAllUsers($pdo) as $user) {
    processUser($user); // too late — full dataset already in memory
}
```

## Better
```php
<?php

declare(strict_types=1);

// Chunked fetching reduces peak memory, but still allocates chunk arrays
function getUsersInChunks(\PDO $pdo, int $size = 1000): iterable
{
    $offset = 0;
    do {
        $stmt = $pdo->prepare('SELECT * FROM users LIMIT :l OFFSET :o');
        $stmt->bindValue(':l', $size,   \PDO::PARAM_INT);
        $stmt->bindValue(':o', $offset, \PDO::PARAM_INT);
        $stmt->execute();
        $rows = $stmt->fetchAll(\PDO::FETCH_ASSOC);
        yield from $rows;
        $offset += $size;
    } while (count($rows) === $size);
}
```

## Best
```php
<?php

declare(strict_types=1);

// File: constant memory regardless of file size
function readLines(string $path): \Generator
{
    $handle = fopen($path, 'r');

    try {
        while (($line = fgets($handle)) !== false) {
            yield rtrim($line, "\n\r");
        }
    } finally {
        fclose($handle); // always released
    }
}

// Database: chunked fetch, yielded row by row
function streamUsers(\PDO $pdo, int $chunkSize = 500): \Generator
{
    $offset = 0;

    do {
        $stmt = $pdo->prepare(
            'SELECT * FROM users ORDER BY id LIMIT :limit OFFSET :offset'
        );
        $stmt->bindValue(':limit',  $chunkSize, \PDO::PARAM_INT);
        $stmt->bindValue(':offset', $offset,    \PDO::PARAM_INT);
        $stmt->execute();

        $rows = $stmt->fetchAll(\PDO::FETCH_ASSOC);

        foreach ($rows as $row) {
            yield $row['id'] => User::fromArray($row); // keyed yield
        }

        $offset += $chunkSize;
    } while (count($rows) === $chunkSize);
}

// Compose generators with yield from
function activeUsers(\PDO $pdo): \Generator
{
    foreach (streamUsers($pdo) as $user) {
        if ($user->isActive()) {
            yield $user;
        }
    }
}

// Usage: one user in memory at a time
foreach (activeUsers($pdo) as $user) {
    exportUser($user);
}
```

## Exceptions / trade-offs
Generators are forward-only and single-pass — you cannot rewind or count them without consuming them. If you need random access, a count, or multiple passes over the same data, an array is more appropriate. Generators also cannot return values from `yield` expressions directly when used with `foreach`; use `Generator::getReturn()` if the return value matters.

## Static-analysis notes
PHPStan and Psalm understand `\Generator` as `iterable`. Annotate return types as `\Generator` (not `iterable`) for tools to verify that `yield` is present and that the key/value types are correct. Use `@return \Generator<int, User, mixed, void>` for full generic annotation.

## Related topics
- [perf-array-functions.md](perf-array-functions.md) — array functions for in-memory collections
- [perf-lazy-loading.md](perf-lazy-loading.md) — deferred initialization of expensive objects
- [error-finally-cleanup.md](error-finally-cleanup.md) — always release resources inside generators with `finally`
