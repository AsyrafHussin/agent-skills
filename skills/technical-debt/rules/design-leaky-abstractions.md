---
title: Leaky Abstractions
impact: HIGH
impactDescription: "Implementation details leak past layer boundaries, blocking change"
tags: abstractions, layers, design
---

## Leaky Abstractions

**Impact: HIGH (Implementation details leak past layer boundaries, blocking change)**

An abstraction leaks when its consumers depend on details it was supposed to hide — ORM models in controllers, framework types in domain logic, HTTP concerns in repositories. Once leaked, the abstraction can no longer be changed independently.

## How to Detect

- Grep for ORM/framework types in inner layers (`Eloquent\Model`, `Request`, `HttpClient`) where they should not appear
- Look for return types like `Builder`, `Collection<Model>`, or `Response` crossing service boundaries

```bash
# Laravel: Eloquent leaking out of repositories
grep -rEn 'extends Model|Eloquent\\Builder' app/Services/ app/Domain/

# Express: Request/Response leaking into services
grep -rEn '\\bRequest\\b|\\bResponse\\b' src/services/
```

## Incorrect

```php
// ❌ Repository returns an Eloquent Builder — controller chains ORM calls
final class OrderRepository
{
    public function forCustomer(int $customerId): Builder
    {
        return Order::query()->where('customer_id', $customerId);
    }
}

// Controller does ORM chaining directly:
$orders = $this->repo->forCustomer($id)
    ->where('status', 'paid')
    ->with(['items', 'shipments'])
    ->orderByDesc('created_at')
    ->paginate(20);
```

**Problems:**
- Repository's promised abstraction ("get orders for customer") is gone — controllers issue arbitrary queries
- Cannot swap Eloquent for another data source without rewriting every controller
- N+1 risk now lives in controllers, not in one auditable place

## Correct

```php
// ✅ Repository returns plain DTOs or a paginated value object — no Builder leak
final class OrderRepository
{
    /** @return Paginated<OrderSummary> */
    public function paidForCustomer(int $customerId, int $page = 1): Paginated
    {
        $query = Order::query()
            ->where('customer_id', $customerId)
            ->where('status', 'paid')
            ->with(['items', 'shipments'])
            ->orderByDesc('created_at');

        return Paginated::from($query->paginate(20, page: $page), OrderSummary::class);
    }
}
```

**Benefits:**
- Controllers receive a stable type; database choice is hidden
- Eager-loading and ordering are owned by the repository (one auditable place)
- Repository can be replaced with an HTTP gateway, gRPC client, or in-memory fake

## Remediation Strategy

- **Effort:** M–L (each leak is local but there are usually many)
- **When to pay down:** When a swap or split is on the roadmap, OR when you find yourself fixing N+1 bugs in multiple controllers — the leak is now causing concrete pain.

Reference: [Joel Spolsky — The Law of Leaky Abstractions](https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/)
