---
title: God Classes
impact: CRITICAL
impactDescription: "One-class kingdoms become merge-conflict and bug magnets"
tags: srp, god-class, refactoring
---

## God Classes

**Impact: CRITICAL (One-class kingdoms become merge-conflict and bug magnets)**

A class with too many responsibilities (often called a "god class") attracts every change in the system. It becomes the file with the most commits, the most authors, and the most bugs — and it blocks parallel work.

## How to Detect

```bash
# Files larger than 300 lines (rough heuristic)
find . -type f \( -name '*.php' -o -name '*.ts' \) -exec wc -l {} \; | \
  awk '$1 > 300 { print }' | sort -rn

# PHP
vendor/bin/phpmd app text codesize  # reports ExcessiveClassLength, TooManyMethods
#   PHPMD defaults are looser (1000 LoC, 25 methods); for stricter 300/15 limits, use a custom
#   ruleset overriding `<property name="minimum" .../>` on those rules.

# Hotspot detection — files changed most often
git log --since='12 months ago' --name-only --pretty=format: | \
  sort | uniq -c | sort -rn | head -20
```

Thresholds: **> 300 lines**, **> 15 public methods**, OR **> 7 dependencies in the constructor**.

## Incorrect

```php
// ❌ OrderService has 820 lines, 27 public methods, depends on 12 services
final class OrderService
{
    public function __construct(
        private OrderRepo $orders,
        private CartRepo $carts,
        private PaymentGateway $payments,
        private TaxCalculator $tax,
        private ShippingService $shipping,
        private InventoryService $inventory,
        private NotificationService $notifications,
        private AnalyticsService $analytics,
        private FraudCheckService $fraud,
        private LoyaltyService $loyalty,
        private InvoiceService $invoices,
        private RefundService $refunds,
    ) {}

    public function create(...)    { /* 80 lines */ }
    public function update(...)    { /* 60 lines */ }
    public function cancel(...)    { /* 70 lines */ }
    public function refund(...)    { /* 90 lines */ }
    public function ship(...)      { /* ... */ }
    public function calculateTax(...) { /* ... */ }
    // ... 21 more methods
}
```

**Problems:**
- Every team that touches orders edits this one file → constant merge conflicts
- Unit tests must mock 12 collaborators just to instantiate it
- A bug in refund logic puts the entire order flow at risk to deploy

## Correct

```php
// ✅ Split by lifecycle stage / responsibility
final class OrderPlacement       { /* create() */ }
final class OrderFulfillment     { /* ship(), markDelivered() */ }
final class OrderCancellation    { /* cancel() */ }
final class OrderRefund          { /* refund() */ }
final class OrderTaxCalculation  { /* calculateTax() */ }

// Each has 2–4 dependencies and 30–100 LoC
```

**Benefits:**
- Teams can work on different stages in parallel without conflicts
- Each class has a focused test suite with minimal mocking
- Bug in refund logic blocks only the refund deploy, not new orders

## Remediation Strategy

- **Effort:** L (almost always — break into multiple PRs)
- **When to pay down:** Identify the highest-churn god class first (`git log` hotspots). Carve off one responsibility per PR — do not attempt a single big-bang refactor.

Reference: [Martin Fowler — Large Class](https://refactoring.com/catalog/extractClass.html)
