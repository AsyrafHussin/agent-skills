---
title: Unbounded Result Sets
impact: HIGH
impactDescription: "One large customer kills latency for everyone"
tags: performance, pagination, database
---

## Unbounded Result Sets

**Impact: HIGH (One large customer kills latency for everyone)**

An endpoint that returns "all" records works fine until one customer has 100,000 of them. At that point, the request times out, exhausts memory, and (in shared-tenancy systems) takes down the database for everyone else. Unbounded result sets are time bombs proportional to your most successful customer.

## How to Detect

```bash
# Find list endpoints / repository methods without LIMIT or pagination
grep -rEn '\\->all\\(|\\->get\\(\\)|findAll|fetchAll' --include='*.php' --include='*.ts'

# Frontend: full-result-set components without "load more" / virtualization
grep -rEn 'map\\(|forEach\\(' --include='*.tsx' src/ | head -50  # review for unbounded lists

# Check actual prod metrics: max rows returned by each endpoint over the last 30 days
# (most APMs / DB monitors expose this)
```

Heuristic: any list endpoint **without an explicit limit** is debt. Any endpoint that returns objects with nested collections (orders → items, users → posts) is doubly so.

## Incorrect

```php
// ❌ Returns every order ever placed by the customer
public function index(Customer $customer) {
    return OrderResource::collection($customer->orders);   // 50,000 rows → 8MB response
}

// ❌ "Export all users" endpoint loads entire table into memory
public function export() {
    $users = User::all();                                  // OOM at scale
    return Excel::download(new UsersExport($users), 'users.xlsx');
}
```

```typescript
// ❌ Frontend asks for everything and filters client-side
const allTransactions = await fetch('/api/transactions').then(r => r.json());
const recent = allTransactions.filter(t => isThisMonth(t.date));   // 100,000 rows → 200,000 ms parse
```

## Correct

```php
// ✅ Cursor-based pagination (preferred for "infinite scroll" / large datasets)
public function index(Customer $customer, Request $request) {
    return OrderResource::collection(
        $customer->orders()->latest()->cursorPaginate(50)
    );
}

// ✅ Streaming export — never loads the full set into memory
public function export() {
    return response()->streamDownload(function () {
        User::query()->orderBy('id')->lazy()->each(function ($user) {
            // write one row at a time
        });
    }, 'users.csv');
}
```

```typescript
// ✅ Server filters; client requests only what it needs
const { data, nextCursor } = await fetch('/api/transactions?since=2026-05-01&limit=50')
  .then(r => r.json());
```

**Benefits:**
- Memory and latency stay bounded regardless of customer size
- Database returns fewer rows over the wire
- Frontend can render results progressively

## Remediation Strategy

- **Effort:** S–M per endpoint (cursor pagination is more invasive than offset; both are mechanical)
- **When to pay down:**
  - **NOW:** any endpoint where the result count is user-controlled and unbounded
  - **NOW:** any export endpoint loading the entire result set into memory
  - **Then:** add a max-result-count assertion in CI for list endpoints
- **Pagination choice:**
  - **Cursor** — best for "next page" UX, large datasets, real-time-ish data (no skip cost)
  - **Offset** — easier to implement, OK for small/medium datasets; expensive at high page numbers
  - **Keyset** — similar to cursor but using a real column (id, created_at)

**Tip:** when retrofitting pagination on a public API, support both old (full response) and new (paginated) shapes during a deprecation window, then remove the unbounded form.

Reference: [Laravel Pagination](https://laravel.com/docs/pagination) · [Slack — Cursor Pagination](https://docs.slack.dev/apis/web-api/pagination) · [Use the Index, Luke — Paging Through Results](https://use-the-index-luke.com/sql/partial-results/fetch-next-page)
