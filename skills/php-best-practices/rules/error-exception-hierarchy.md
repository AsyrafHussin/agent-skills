---
title: Keep Exception Hierarchies Shallow and Purposeful
impact: HIGH
impactDescription: Robust error handling and exception design
tags: php, error-handling, exceptions
---

# Keep Exception Hierarchies Shallow and Purposeful

## Why it matters
A six-level exception tree feels architectural but creates real costs: callers must know which level to catch, new developers must read the whole tree before writing a single `catch`, and renaming a node is a breaking change. Hierarchies only pay off when callers actually catch at an intermediate level.

## Rule
Use at most three levels: one root domain exception, a small number of semantically distinct families (not-found, validation, infrastructure), and concrete leaf types where callers need them. Add a level only when a real catch site needs it.

## Bad
```php
<?php

declare(strict_types=1);

// Flat — no hierarchy, forces callers to catch each type individually
class UserNotFoundException extends \Exception {}
class OrderNotFoundException extends \Exception {}
class ProductNotFoundException extends \Exception {}
class InvalidEmailException extends \Exception {}
class InvalidPriceException extends \Exception {}
class DatabaseConnectionException extends \Exception {}
class ApiTimeoutException extends \Exception {}
```

## Better
```php
<?php

declare(strict_types=1);

// Better: groups exist, but rooted at \Exception not \RuntimeException
class AppException extends \Exception {}
class NotFoundException extends AppException {}
class UserNotFoundException extends NotFoundException {}
class ValidationException extends AppException {}
class InfrastructureException extends AppException {}
```

## Best
```php
<?php

declare(strict_types=1);

// Root — extend RuntimeException (unchecked, no forced catch)
class AppException extends \RuntimeException {}

// Family: not-found — callers can catch the family or a leaf
class NotFoundException extends AppException
{
    public function __construct(
        private readonly string $entity,
        private readonly string|int $id,
    ) {
        parent::__construct("{$entity} not found: {$id}");
    }

    public function entity(): string { return $this->entity; }
}

final class UserNotFoundException extends NotFoundException
{
    public function __construct(string|int $id)
    {
        parent::__construct('User', $id);
    }
}

final class OrderNotFoundException extends NotFoundException
{
    public function __construct(string|int $id)
    {
        parent::__construct('Order', $id);
    }
}

// Family: validation
final class ValidationException extends AppException
{
    /** @param array<string, string> $errors */
    public function __construct(private readonly array $errors = [])
    {
        parent::__construct('Validation failed');
    }

    /** @return array<string, string> */
    public function errors(): array { return $this->errors; }
}

// Family: infrastructure — both DB and API resolve to 503
class InfrastructureException extends AppException {}
final class DatabaseException extends InfrastructureException {}
final class ExternalApiException extends InfrastructureException {}

// Callers catch at the level they care about
try {
    $service->processOrder($data);
} catch (NotFoundException $e) {
    return response()->json(['error' => $e->getMessage()], 404);
} catch (ValidationException $e) {
    return response()->json(['errors' => $e->errors()], 422);
} catch (InfrastructureException $e) {
    $logger->error($e->getMessage());
    return response()->json(['error' => 'Service unavailable'], 503);
}
```

## Exceptions / trade-offs
Cross-cutting concerns (e.g. a `RetryableException` interface) are better expressed as interfaces than as hierarchy levels. If only one leaf exists under a family, skip the intermediate class until a second leaf is needed.

## Static-analysis notes
PHPStan and Psalm both understand inheritance-based catch broadening. Marking leaf classes `final` prevents accidental extension and lets tools confirm exhaustive coverage.

## Related topics
- [error-custom-exceptions.md](error-custom-exceptions.md) — when to create a custom exception at all
- [error-try-catch-specific.md](error-try-catch-specific.md) — catching at the right level
