---
title: Use Prepared Statements for All SQL
impact: CRITICAL
impactDescription: Security best practices and input safety
tags: php, security, validation
---

# Use Prepared Statements for All SQL

## Why it matters
SQL injection is the most reliably exploitable web vulnerability, with documented impact ranging from data exfiltration to full server compromise. It is caused by a single pattern: user input concatenated into a SQL string. Prepared statements separate code from data at the protocol level, making injection structurally impossible for parameterized values.

## Rule
Use prepared statements with bound parameters for every query that includes a variable value. No string concatenation with user input. No exceptions for "quick" queries, admin endpoints, or internal APIs.

## Bad
```php
<?php

declare(strict_types=1);

// Classic injection — ' OR '1'='1 returns all rows
$email = $_POST['email'];
$db->query("SELECT * FROM users WHERE email = '$email'");

// Interpolation is identical in risk
$id = $_GET['id'];
$db->query("SELECT * FROM orders WHERE id = $id");

// sprintf does not escape; it just formats the string
$sql = sprintf("SELECT * FROM users WHERE name = '%s'", $name);

// Type-casting is fragile — one forgotten cast is an injection
$id = (int) $_GET['id'];
```

## Better
```php
<?php

declare(strict_types=1);

// Named parameters — correct, but EMULATE_PREPARES may be on by default
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => $email]);
```

## Best
```php
<?php

declare(strict_types=1);

// Configure PDO for real server-side prepared statements
$pdo = new \PDO($dsn, $user, $pass, [
    \PDO::ATTR_ERRMODE            => \PDO::ERRMODE_EXCEPTION,
    \PDO::ATTR_DEFAULT_FETCH_MODE => \PDO::FETCH_ASSOC,
    \PDO::ATTR_EMULATE_PREPARES   => false, // real preparation, not emulation
]);

// Named parameters — preferred for readability
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email AND active = :active');
$stmt->execute(['email' => $email, 'active' => 1]);
$user = $stmt->fetch();

// Explicit type binding for non-string values
$stmt = $pdo->prepare(
    'SELECT * FROM orders WHERE total > :min LIMIT :limit OFFSET :offset'
);
$stmt->bindValue(':min',    $minAmount, \PDO::PARAM_STR);
$stmt->bindValue(':limit',  $pageSize,  \PDO::PARAM_INT);
$stmt->bindValue(':offset', $offset,    \PDO::PARAM_INT);
$stmt->execute();

// INSERT
$stmt = $pdo->prepare(
    'INSERT INTO users (name, email, created_at) VALUES (:name, :email, NOW())'
);
$stmt->execute(['name' => $name, 'email' => $email]);

// Column/table names CANNOT be parameterized — whitelist them
$allowed = ['name', 'email', 'created_at', 'price'];
$column  = in_array($sortBy, $allowed, true) ? $sortBy : 'id';
$stmt    = $pdo->prepare("SELECT * FROM users ORDER BY {$column} ASC");
$stmt->execute();
```

## Exceptions / trade-offs
Column names and table names cannot be bound as parameters — they must be whitelisted from a hard-coded list. Dynamic `IN (?, ?, ?)` clauses require constructing the placeholder list programmatically, which is mechanical but necessary; use a loop and `implode`. Never bind a comma-separated string like `'1,2,3'` into a single placeholder inside `IN (?)`; that is not equivalent to per-item binding and silently breaks filtering.

## Static-analysis notes
Psalm's taint analysis (`--taint-analysis`) and phpcs rules such as `WordPress.DB.PreparedSQL` can identify concatenated SQL. PHPStan has no built-in SQL injection detection without a plugin.

## Security notes
`PDO::ATTR_EMULATE_PREPARES = false` forces the database driver to use real server-side statement preparation. Emulation performs client-side escaping, which has had historical bypass vulnerabilities in specific encoding combinations. Disable it.

## Related topics
- [sec-input-validation.md](sec-input-validation.md) — whitelist column names as part of input validation
- [sec-output-escaping.md](sec-output-escaping.md) — context-specific protection; SQL is its own context
