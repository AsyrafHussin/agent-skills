---
title: Create Custom Exceptions Only When Callers Need to React Differently
impact: HIGH
impactDescription: Robust error handling and exception design
tags: php, error-handling, exceptions
---

# Create Custom Exceptions Only When Callers Need to React Differently

## Why it matters
Throwing a generic `\RuntimeException` with a message string forces callers to match error text — fragile, untypeable, and invisible to static analysis. But creating twenty exception classes for decorative taxonomy wastes maintenance overhead with no engineering return. The breakeven is: can a caller do something meaningfully different in a `catch` block for this type?

## Rule
Create a custom exception class when: (a) callers need to catch it separately from other failures, or (b) the exception must carry domain-specific data that a message string cannot type-safely express. Otherwise use a standard SPL exception.

## Bad
```php
<?php

declare(strict_types=1);

final class UserService
{
    public function register(array $data): User
    {
        // All failures look identical to callers
        if (empty($data['email'])) {
            throw new \Exception('Email is required');
        }
        if ($this->repository->findByEmail($data['email'])) {
            throw new \Exception('Email already exists');
        }
        if (!$this->gateway->charge($data['amount'])) {
            throw new \Exception('Payment failed');
        }
        return $this->repository->create($data);
    }
}
```

## Better
```php
<?php

declare(strict_types=1);

// Custom classes exist but carry no data — a message string would do
final class EmailRequiredException extends \InvalidArgumentException {}
final class EmailAlreadyExistsException extends \RuntimeException {}
final class PaymentFailedException extends \RuntimeException {}
```

## Best
```php
<?php

declare(strict_types=1);

// Each exception carries typed context the caller can inspect
final class ValidationException extends \RuntimeException
{
    /** @param array<string, string> $errors */
    public function __construct(
        private readonly array $errors,
        string $message = 'Validation failed',
    ) {
        parent::__construct($message);
    }

    /** @return array<string, string> */
    public function errors(): array
    {
        return $this->errors;
    }
}

final class DuplicateEmailException extends \RuntimeException
{
    public function __construct(public readonly string $email)
    {
        parent::__construct("Email already registered: {$email}");
    }
}

final class PaymentFailedException extends \RuntimeException
{
    public function __construct(
        public readonly string $reason,
        public readonly ?string $gatewayCode = null,
        ?\Throwable $previous = null,
    ) {
        parent::__construct("Payment declined: {$reason}", previous: $previous);
    }
}

// Caller can react precisely
try {
    $service->register($data);
} catch (ValidationException $e) {
    return response()->json(['errors' => $e->errors()], 422);
} catch (DuplicateEmailException $e) {
    return response()->json(['error' => 'Email taken'], 409);
} catch (PaymentFailedException $e) {
    return response()->json(['error' => 'Payment failed', 'code' => $e->gatewayCode], 402);
}

// Throwing side — each exception carries typed domain data
final class UserService
{
    public function register(array $data): User
    {
        if (empty($data['email'])) {
            throw new ValidationException(['email' => 'Email is required']);
        }
        if ($this->repository->findByEmail($data['email'])) {
            throw new DuplicateEmailException($data['email']);
        }
        if (!$this->gateway->charge($data['amount'])) {
            throw new PaymentFailedException('Card declined', $this->gateway->lastCode());
        }
        return $this->repository->create($data);
    }
}
```

## Exceptions / trade-offs
If the only consumer of the exception is a global handler that logs and returns a 500, a named class adds no value. Prefer `throw new \RuntimeException(...)` in that case. Do not create exception classes speculatively — wait until the catch site exists.

## Static-analysis notes
PHPStan and Psalm verify that `catch` blocks reference real exception classes and that accessed properties exist. `readonly` constructor properties are fully understood by both tools.

## Related topics
- [error-try-catch-specific.md](error-try-catch-specific.md) — catching exceptions narrowly
- [error-exception-hierarchy.md](error-exception-hierarchy.md) — grouping exceptions into families
