---
title: Missing Caching Opportunities
impact: HIGH
impactDescription: "Repeated work on every request — predictable latency you're paying for indefinitely"
tags: performance, caching, redis, http-cache
---

## Missing Caching Opportunities

**Impact: HIGH (Repeated work on every request — predictable latency you're paying for indefinitely)**

Caching is the highest-ROI performance work for read-heavy systems: a single Redis lookup replaces a 200ms aggregation query, and an HTTP cache header lets the browser skip the round-trip entirely. Missing caching is invisible — the system "works" — but every page load pays the full computation cost.

## How to Detect

Look for these signals in any read-heavy path:

1. **Expensive aggregations recomputed per request** (dashboard counters, reports, leaderboards)
2. **External API calls without caching** (currency rates, geolocation, third-party catalogs)
3. **Static-ish responses with no `Cache-Control` headers** (asset metadata, config, public lists)
4. **Database queries that join 5+ tables to return the same shape repeatedly**
5. **Same query fired by N concurrent requests, no request coalescing**

```bash
# Look at HTTP response headers — missing Cache-Control on static-ish endpoints
curl -sI https://your.app/api/categories | grep -iE 'cache-control|etag|last-modified'

# Find APIs that re-compute the same shape across requests
# (look in your APM for endpoints with consistently high mean latency + low variance)

# Laravel — endpoints that hit expensive accessors / relations without remember()
grep -rEn '\\->withCount\\(|\\->withSum\\(' app/Http/Controllers/ | head
```

## Incorrect

```php
// ❌ Recomputed on every request — even though categories change ~once a week
public function categories() {
    $categories = Category::query()
        ->withCount('products')
        ->with(['parent', 'translations'])
        ->orderBy('sort_order')
        ->get();

    return CategoryResource::collection($categories);
}

// ❌ Third-party rate fetched per request — 200ms latency on every checkout
public function checkout(Order $order) {
    $rate = Http::get('https://api.fx.example.com/rates/USD-MYR')->json('rate');
    return ['total' => $order->total * $rate];
}

// ❌ No HTTP caching → browser revalidates on every navigation
return response()->json($publicConfig);
```

**Problems:**
- Database scans for `categories` happen N times per second across the whole fleet
- Every checkout pays 200ms for an FX rate that updates hourly
- Browser fetches `publicConfig` on every page load even though it changes daily

## Correct

```php
// ✅ Tag-keyed Redis cache with explicit invalidation
// Note: Cache::tags() only works with the `redis` or `memcached` driver.
// On `file` / `database` / `array` stores it throws BadMethodCallException —
// fall back to plain keys + manual invalidation on those drivers.
public function categories() {
    $categories = Cache::tags(['categories'])->remember(
        'categories:tree:v2',
        now()->addHour(),
        fn () => Category::query()
            ->withCount('products')
            ->with(['parent', 'translations'])
            ->orderBy('sort_order')
            ->get(),
    );
    return CategoryResource::collection($categories);
}

// In Category::saved / Category::deleted listeners:
//   Cache::tags(['categories'])->flush();
```

```php
// ✅ External API result cached for an hour
$rate = Cache::remember('fx:USD-MYR', now()->addHour(), function () {
    return Http::get('https://api.fx.example.com/rates/USD-MYR')->json('rate');
});
```

```php
// ✅ HTTP cache headers for safely-cacheable public responses
return response()->json($publicConfig)
    ->header('Cache-Control', 'public, max-age=3600, stale-while-revalidate=86400')
    ->header('ETag', md5(json_encode($publicConfig)));
```

**Benefits:**
- Mean latency drops orders of magnitude for cacheable endpoints
- Origin server, database, and third-party APIs all see reduced load
- Browser short-circuits revalidation for unchanged resources

## Remediation Strategy

- **Effort:**
  - **S** — add `remember()` / HTTP `Cache-Control` to a single endpoint
  - **M** — design a cache-key scheme + invalidation strategy for a domain
  - **L** — distributed caching with proper invalidation across services
- **When to pay down:**
  - **NOW:** any endpoint hitting an expensive query AND showing high traffic AND data is read-mostly
  - **Then:** look at top-10 requests by total time in APM; cache the easy wins first
- **Layer order (cheapest first):**
  1. HTTP cache headers (browser does the work)
  2. CDN / edge cache (`Cache-Control: public, s-maxage=N`)
  3. Application memory cache (per-process, fastest, no network)
  4. Redis / Memcached (shared across processes, sub-ms)
  5. Database query cache / materialized views (last resort)

**Anti-patterns:**
- **Caching everything by default** — cache invalidation is hard; cache only what hurts
- **TTL-only invalidation when freshness matters** — combine with event-based busts (`Cache::flush` on writes)
- **Caching personalized data with a public key** — leaks one user's data to another
- **Caching error responses indefinitely** — always exclude 4xx/5xx from caches

Reference: [Laravel — Cache](https://laravel.com/docs/cache) · [MDN — HTTP Caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching) · [RFC 9111 — HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111)
