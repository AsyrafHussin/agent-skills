---
title: Validate at Every Boundary
impact: CRITICAL
impactDescription: Security best practices and input safety
tags: php, security, validation
---

# Validate at Every Boundary

## Why it matters
Unvalidated input is the root cause of SQL injection, XSS, path traversal, and command injection. A single call path that skips validation — even an internal API endpoint or an admin-only form — is sufficient for a breach. Validation performed only on the frontend is irrelevant: HTTP requests bypass the browser entirely.

## Rule
Validate at every trust boundary, normalize exactly once, and narrow types as early as possible. Reject unknown input by default; allow known-good values explicitly.
Choose validation and normalization for the next sink: persistence and business logic need raw normalized values, while HTML safety belongs at the final output boundary.

## Bad
```php
<?php

declare(strict_types=1);

// Raw superglobals used directly — no validation, no typing
$name  = $_POST['name'];
$email = $_POST['email'];
$age   = $_POST['age'];
$sort  = $_GET['sort']; // injected into query below

$db->query("SELECT * FROM items ORDER BY {$sort}");
```

## Better
```php
<?php

declare(strict_types=1);

// Some validation, but inconsistent — age cast but never range-checked,
// sort whitelisted but name length unchecked
$email = filter_var($_POST['email'] ?? '', FILTER_VALIDATE_EMAIL);
$age   = (int) ($_POST['age'] ?? 0);

$allowed = ['id', 'name', 'price'];
$sort = in_array($_GET['sort'] ?? '', $allowed, true) ? $_GET['sort'] : 'id';
```

## Best
```php
<?php

declare(strict_types=1);

/**
 * Validates and normalizes a user-registration payload.
 * Returns typed, safe values — or throws ValidationException.
 *
 * @param  array<string, mixed> $input
 * @return array{name: string, email: string, age: int}
 */
function parseRegistration(array $input): array
{
    $email = filter_var($input['email'] ?? '', FILTER_VALIDATE_EMAIL);
    if ($email === false) {
        throw new ValidationException(['email' => 'Invalid email address']);
    }

    $name = trim((string) ($input['name'] ?? ''));
    if ($name === '' || mb_strlen($name) > 100) {
        throw new ValidationException(['name' => 'Name must be 1–100 characters']);
    }

    $age = filter_var(
        $input['age'] ?? null,
        FILTER_VALIDATE_INT,
        ['options' => ['min_range' => 1, 'max_range' => 150]],
    );
    if ($age === false) {
        throw new ValidationException(['age' => 'Age must be between 1 and 150']);
    }

    return ['name' => $name, 'email' => $email, 'age' => $age];
}

// Dynamic column names: whitelist, never parameterize the name itself
function buildListQuery(PDO $pdo, array $input): \PDOStatement
{
    $allowed = ['id', 'name', 'created_at', 'price'];
    $sort    = in_array($input['sort'] ?? '', $allowed, true)
        ? $input['sort']
        : 'id';

    $page   = max(1, (int) ($input['page'] ?? 1));
    $limit  = 20;
    $offset = ($page - 1) * $limit;

    $stmt = $pdo->prepare(
        "SELECT * FROM items ORDER BY {$sort} LIMIT :limit OFFSET :offset"
    );
    $stmt->bindValue(':limit',  $limit,  \PDO::PARAM_INT);
    $stmt->bindValue(':offset', $offset, \PDO::PARAM_INT);

    return $stmt;
}
```

## Exceptions / trade-offs
Do not validate inside domain models or services — that leaks boundary concerns inward. Validate at the HTTP controller or console command boundary, then pass typed value objects downstream. For bulk imports, consider validating and collecting all errors before aborting so the user sees all problems at once. Do not persist pre-escaped HTML-safe values just because they arrived from a browser form; store normalized raw data and escape only when rendering.

## Static-analysis notes
Psalm's `@param-type` and PHPStan's `array-shape` can enforce the shape of validated arrays. Return a typed DTO rather than an array for the strongest guarantees.

## Security notes
Blacklisting (stripping "bad" characters) is weaker than whitelisting. `filter_var` with `FILTER_VALIDATE_*` flags returns `false` on failure — check that explicitly; `null` and `0` are also falsy but valid in some contexts, so use strict comparison (`=== false`).

## Related topics
- [sec-sql-prepared.md](sec-sql-prepared.md) — prepared statements prevent injection from dynamic SQL
- [sec-output-escaping.md](sec-output-escaping.md) — validation and escaping are complementary, not alternatives
- [type-strict-mode.md](type-strict-mode.md) — `declare(strict_types=1)` prevents silent type coercion
