---
title: Asymmetric Visibility
impact: CRITICAL
impactDescription: Modern PHP 8.x language features
tags: php, modern-php, php8
---

# Asymmetric Visibility

## Why it matters
Encapsulating a property behind a private field plus a trivial getter method is boilerplate that adds noise without adding value. PHP 8.4 asymmetric visibility lets a property be publicly readable but only writable by the owning class, eliminating the getter entirely without exposing the write path.

## Rule
Use `public private(set)` for properties that must be readable everywhere but writable only within the owning class. This replaces trivial getter methods without sacrificing encapsulation.

## Bad
```php
<?php

declare(strict_types=1);

final class Order
{
    private string $status;
    private int $total;

    public function __construct(string $status, int $total)
    {
        $this->status = $status;
        $this->total  = $total;
    }

    public function getStatus(): string { return $this->status; }
    public function getTotal(): int     { return $this->total; }

    public function cancel(): void
    {
        $this->status = 'cancelled';
    }
}

// Reading requires calling getters everywhere
echo $order->getStatus();
echo $order->getTotal();
```

## Better
```php
<?php

declare(strict_types=1);

// readonly removes the getter boilerplate, but prevents internal mutation
final class Order
{
    public function __construct(
        public readonly string $status,
        public readonly int $total,
    ) {}

    // Cannot implement cancel() — readonly properties cannot be reassigned
}
```

## Best
```php
<?php

declare(strict_types=1);

final class Order
{
    public private(set) string $status;
    public private(set) int $total;

    public function __construct(string $status, int $total)
    {
        $this->status = $status;
        $this->total  = $total;
    }

    public function cancel(): void
    {
        // Internal write is allowed
        $this->status = 'cancelled';
    }
}

$order = new Order('pending', 5000);

echo $order->status;  // public read — no getter needed
// $order->status = 'x';  // Fatal error — private(set) blocks external writes
$order->cancel();     // internal mutation still works
```

## Exceptions / trade-offs
When a property is **truly immutable** after construction, `readonly` is still the cleaner signal — it communicates the invariant more strongly than `private(set)`. When a property needs external write access, use `public`. Asymmetric visibility fills the gap between those two extremes.

## Static-analysis notes
PHPStan and Psalm 8.4+ understand asymmetric visibility and report write-access violations in static analysis. IDEs show write-access errors at edit time, before running analysis.

## Version notes
`PHP 8.4+`

## Related topics
- [modern-readonly-properties.md](modern-readonly-properties.md) — for truly immutable properties
- [modern-property-hooks.md](modern-property-hooks.md) — for properties with access logic
