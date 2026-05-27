---
title: Avoid Global Variables
impact: MEDIUM
impactDescription: Runtime performance and memory efficiency
tags: php, performance, optimization
---

# Avoid Global Variables

## Why it matters
Global variables create hidden channels between otherwise unrelated parts of a program. Any function can read or overwrite a global at any point, making behavior dependent on execution order rather than explicit inputs. This destroys testability (you cannot inject a test double), makes static analysis unreliable, and turns debugging into archaeology through call stacks looking for where the value was last mutated.

## Rule
Avoid global variables and the `global` keyword entirely. Pass dependencies explicitly through constructors or function parameters. A class should declare everything it needs in its constructor — visible, typed, and replaceable.

## Bad
```php
<?php

declare(strict_types=1);

// Shared mutable state — any file can change these at any time
$db     = new \PDO('mysql:host=localhost;dbname=app', 'root', '');
$config = ['debug' => true, 'cache_ttl' => 3600];
$logger = new FileLogger('/var/log/app.log');

class UserService
{
    public function find(int $id): ?array
    {
        global $db; // hidden dependency; untestable without real DB
        $stmt = $db->prepare('SELECT * FROM users WHERE id = ?');
        $stmt->execute([$id]);
        return $stmt->fetch() ?: null;
    }

    public function isDebug(): bool
    {
        global $config; // another hidden dependency
        return $config['debug'];
    }
}
```

## Better
```php
<?php

declare(strict_types=1);

// Static property — still shared mutable state, but scoped to the class
class UserService
{
    private static ?\PDO $db = null;

    public static function setDb(\PDO $pdo): void
    {
        self::$db = $pdo;
    }

    public function find(int $id): ?array
    {
        $stmt = self::$db->prepare('SELECT * FROM users WHERE id = ?');
        $stmt->execute([$id]);
        return $stmt->fetch() ?: null;
    }
}
```

## Best
```php
<?php

declare(strict_types=1);

// All dependencies declared in the constructor — explicit, typed, injectable
final class UserService
{
    public function __construct(
        private readonly \PDO    $db,
        private readonly Config  $config,
        private readonly LoggerInterface $logger,
    ) {}

    public function find(int $id): ?User
    {
        $stmt = $this->db->prepare('SELECT * FROM users WHERE id = ?');
        $stmt->execute([$id]);
        $row  = $stmt->fetch(\PDO::FETCH_ASSOC);

        if ($row === false) {
            return null;
        }

        $this->logger->debug('User fetched', ['id' => $id]);
        return User::fromArray($row);
    }

    public function isDebug(): bool
    {
        return $this->config->get('debug', false);
    }
}

// Wire dependencies once at the composition root (bootstrap / DI container)
$service = new UserService($pdo, $config, $logger);

// Unit test: swap real dependencies for doubles
$service = new UserService(
    $mockPdo,
    new Config(['debug' => false]),
    new NullLogger(),
);
```

## Exceptions / trade-offs
PHP superglobals (`$_SERVER`, `$_ENV`, `$_GET`, `$_POST`) cannot be avoided at the framework boundary — but they should be read once, at entry, and converted into typed value objects before passing deeper into the application. Never access `$_GET` or `$_POST` inside a service class.

## Static-analysis notes
PHPStan detects access to undefined variables and can be configured to forbid `global` declarations via a custom rule. Psalm's `no-undefined-variable` mode catches global use. Both tools can infer constructor injection correctly from typed properties.

## Related topics
- [solid-dip.md](solid-dip.md) — dependency inversion is the design principle behind constructor injection
- [perf-lazy-loading.md](perf-lazy-loading.md) — lazy-load expensive dependencies without resorting to global state
- [testing-best-practices](../../../testing-best-practices/) — testability depends on injectable dependencies
