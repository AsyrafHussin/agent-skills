---
title: N+1 Query Patterns
impact: HIGH
impactDescription: "Linear request → quadratic database load; latency scales with data, not traffic"
tags: performance, database, n-plus-one
---

## N+1 Query Patterns

**Impact: HIGH (Linear request → quadratic database load; latency scales with data, not traffic)**

An N+1 happens when fetching a list of N records issues 1 query for the list + N follow-up queries for each row's relations. Latency looks fine in dev (small N) and explodes in prod (large N). N+1 is the single most common database debt in ORM-driven codebases.

## How to Detect

```bash
# Laravel: Telescope panel "Queries" — sort by request, count per page
php artisan telescope:install

# Laravel: detect N+1s in dev by failing on excessive queries
composer require beyondcode/laravel-query-detector --dev

# Node / TypeScript: enable query logging in your ORM (Prisma `log: ['query']`,
# TypeORM `logging: 'all'`, Drizzle `logger: true`) and count queries per request.

# Heuristic: > ~10 queries for a typical list endpoint is suspect.
```

Pattern signal: query count grows with result count instead of staying constant.

## Incorrect

```php
// ❌ Laravel: relation accessed inside a loop → one query per order
$orders = Order::where('status', 'paid')->get();        // 1 query

foreach ($orders as $order) {
    $customer = $order->customer;                       // 1 query each (N more)
    $shipping = $order->shipments;                      // 1 query each (another N)
    echo "{$customer->name}: " . $shipping->count();
}
// Total: 1 + 2N queries for 100 orders → 201 queries
```

```typescript
// ❌ TypeORM equivalent
const users = await userRepo.find();                    // 1 query
for (const user of users) {
  const orders = await user.orders;                     // N queries (lazy relation)
  console.log(user.email, orders.length);
}
```

**Problems:**
- A list endpoint that returns 200 rows takes 401 round-trips to the database
- Latency is invisible at low traffic; suddenly catastrophic with real data
- Database connection pool saturates; queue depth spikes for unrelated requests

## Correct

```php
// ✅ Eager-load with `with()` — fixed query count regardless of N
$orders = Order::where('status', 'paid')
    ->with(['customer', 'shipments'])
    ->get();   // 3 queries total: orders, customers, shipments

foreach ($orders as $order) {
    echo "{$order->customer->name}: " . $order->shipments->count();
}
```

```typescript
// ✅ Specify relations in the find call
const users = await userRepo.find({ relations: { orders: true } });
```

For large result sets, also paginate:

```php
$orders = Order::with(['customer', 'shipments'])
    ->where('status', 'paid')
    ->cursorPaginate(50);   // memory + DB bounded
```

**Benefits:**
- Query count becomes constant per request, regardless of result size
- Latency is predictable in production
- Connection pool stays healthy under load

## Remediation Strategy

- **Effort:** S per endpoint (add `with(...)` or equivalent eager-load)
- **When to pay down:**
  - **NOW:** any endpoint that emits > 10× more queries than its result count
  - **Then:** add a CI test that asserts query counts on critical endpoints
- **Detection workflow:**
  1. Install Telescope / Bullet / Silk
  2. Walk the top 10 most-hit endpoints in a representative env
  3. Sort by query count per request — N+1s are at the top
  4. Add `with(...)` and re-measure

**Anti-pattern:** "let's add a cache" before fixing N+1. Caching hides the problem but doesn't fix it — the first request after expiry still does 201 queries.

Reference: [Laravel Eager Loading](https://laravel.com/docs/eloquent-relationships#eager-loading) · [beyondcode/laravel-query-detector](https://github.com/beyondcode/laravel-query-detector)
