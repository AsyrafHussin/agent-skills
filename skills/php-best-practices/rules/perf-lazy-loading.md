---
title: Lazy-Load Only Genuinely Expensive Dependencies
impact: MEDIUM
impactDescription: Runtime performance and memory efficiency
tags: php, performance, optimization
---

# Lazy-Load Only Genuinely Expensive Dependencies

## Why it matters
A constructor that queries the database, initializes a PDF renderer, and loads configuration from disk makes the class slow to instantiate for every caller — including unit tests, health checks, and code paths that never touch those resources. But over-engineering lazy loading adds null-tracking, initializer methods, and indirection that obscures intent without measurable benefit for cheap dependencies.

## Rule
Lazy-load only dependencies or data whose initialization cost is real and measurable: heavy I/O, large dataset retrieval, or resource-intensive object construction. Use `??=` for simple lazy initialization. Inject cheap services eagerly via the constructor.

## Bad
```php
<?php

declare(strict_types=1);

final class ReportService
{
    private array $allUsers;
    private array $allOrders;
    private PdfRenderer $pdf;

    public function __construct(
        private readonly UserRepository  $users,
        private readonly OrderRepository $orders,
    ) {
        // Runs for every instantiation — even for getUserCount()
        $this->allUsers  = $this->users->findAll();   // SELECT * FROM users
        $this->allOrders = $this->orders->findAll();  // SELECT * FROM orders
        $this->pdf       = new PdfRenderer();         // loads fonts, templates
    }

    public function getUserCount(): int
    {
        return count($this->allUsers);
    }
}
```

## Better
```php
<?php

declare(strict_types=1);

final class ReportService
{
    private ?array $allOrders = null;
    private ?PdfRenderer $pdf = null;

    public function __construct(
        private readonly UserRepository  $users,
        private readonly OrderRepository $orders,
    ) {}

    public function getUserCount(): int
    {
        // Uses a targeted query instead of loading all users
        return $this->users->count();
    }

    public function generateReport(): string
    {
        $this->allOrders ??= $this->orders->findAll();
        $this->pdf       ??= new PdfRenderer();

        return $this->pdf->render($this->allOrders);
    }
}
```

## Best
```php
<?php

declare(strict_types=1);

final class ReportService
{
    private ?PdfRenderer $pdf = null;

    public function __construct(
        private readonly UserRepository  $users,
        private readonly OrderRepository $orders,
    ) {}

    public function getUserCount(): int
    {
        return $this->users->count(); // single COUNT query, no data transfer
    }

    public function generateReport(): string
    {
        // Orders loaded only when generateReport() is actually called
        $orders = $this->orders->findAll();

        return $this->getPdf()->render($orders);
    }

    // PdfRenderer is expensive (font loading, template compilation)
    // Lazy-initialize with ??= — created once, reused on repeated calls
    private function getPdf(): PdfRenderer
    {
        return $this->pdf ??= new PdfRenderer();
    }
}

// Pattern: closure-based lazy value for a standalone computation
final class LazyValue
{
    private mixed $value = null;
    private bool $initialized = false;

    public function __construct(private readonly \Closure $initializer) {}

    public function get(): mixed
    {
        if (!$this->initialized) {
            $this->value       = ($this->initializer)();
            $this->initialized = true;
        }
        return $this->value;
    }
}
```

## Exceptions / trade-offs
PHP 8.4 introduces lazy object proxies via `ReflectionClass::newLazyProxy()` / `newLazyGhost()`, which can defer the entire construction of an object until first property access. This is the correct tool for DI containers that must instantiate objects at graph-build time but only use a subset at runtime.

Do not lazy-load something that is always needed — you pay the complexity cost without the benefit. Prefer passing an already-constructed dependency through the constructor (DI) over lazy initialization inside the class.

## Static-analysis notes
PHPStan tracks nullable property types and can warn if you access `$this->pdf` without a null check. Using `??=` eliminates the null check concern inside the initializer method; PHPStan understands this operator.

## Version notes
`??=` (null coalescing assignment) — PHP 7.4+
Lazy objects (`ReflectionClass::newLazyProxy`) — PHP 8.4+

## Related topics
- [perf-generators.md](perf-generators.md) — deferred data loading for large sequences
- [perf-avoid-globals.md](perf-avoid-globals.md) — inject dependencies rather than pulling them lazily from global state
- [solid-dip.md](solid-dip.md) — dependency inversion; inject abstractions rather than constructing concretions
