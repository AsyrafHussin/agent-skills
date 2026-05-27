---
title: Typed Class Constants
impact: CRITICAL
impactDescription: Modern PHP 8.x language features
tags: php, modern-php, php8
---

# Typed Class Constants

## Why it matters
An untyped class constant is an implicit `mixed` — a child class can silently override it with a value of a different type, breaking code that relies on the constant's type contract. Typed constants enforce the contract at class-load time and give static analysis something concrete to verify.

## Rule
Type class constants whenever the constant's type contract matters or the constant can be overridden in subclasses. This is cheap to add and prevents a whole class of silent type-coercion bugs.

## Bad
```php
<?php

declare(strict_types=1);

class PaymentStatus
{
    const PENDING  = 'pending';
    const COMPLETE = 'complete';
}

class LegacyPaymentStatus extends PaymentStatus
{
    const PENDING = 1;  // silently changed to int — callers expecting string break at runtime
}
```

## Better
```php
<?php

declare(strict_types=1);

class PaymentStatus
{
    const string PENDING  = 'pending';
    const string COMPLETE = 'complete';
}

class ModernPaymentStatus extends PaymentStatus
{
    // const string PENDING = 1;  // Fatal error — type mismatch caught at class load
}
```

## Best
```php
<?php

declare(strict_types=1);

interface HasStatus
{
    const string PENDING  = 'pending';
    const string COMPLETE = 'complete';
    const string FAILED   = 'failed';
}

final class Order implements HasStatus
{
    public function isPending(): bool
    {
        return $this->status === self::PENDING;  // type guaranteed: always string
    }
}
```

## Exceptions / trade-offs
Constants whose type is obvious from context and that are never overridden do not urgently need explicit types — but adding types is cheap and worth doing. The main payoff comes in class hierarchies where subclasses can override constants. For `enum` values, use actual `enum` types instead of typed constants.

## Static-analysis notes
PHPStan and Psalm validate typed constants and report type mismatches in subclasses as hard errors. Type mismatches that would previously surface only at runtime become CI failures.

## Version notes
`PHP 8.3+`

## Related topics
- [modern-enums.md](modern-enums.md) — for sets of named values, enums are a stronger alternative
