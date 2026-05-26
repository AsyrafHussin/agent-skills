# Sections

This file defines the section ordering, severity, and summary used by this review lens.

---

## 1. Algorithmic Complexity (perf-algorithmic-complexity)

**Impact:** CRITICAL
**Description:** Complexity growth, nested loops, and expensive recomputation.

## 2. Database Performance (perf-database-performance)

**Impact:** CRITICAL
**Description:** N+1 queries, indexes, joins, and query-shape efficiency.

## 3. Network I/O (perf-network-io)

**Impact:** HIGH
**Description:** Batching, pagination, parallelism, timeouts, and unnecessary round trips.

## 4. Memory Management (perf-memory-management)

**Impact:** HIGH
**Description:** Leaks, retention, object copying, and unbounded collections.

## 5. Caching Strategy (perf-caching-strategy)

**Impact:** MEDIUM
**Description:** Cacheability, invalidation, key design, and cold-path cost.

## 6. Concurrency (perf-concurrency)

**Impact:** HIGH
**Description:** Blocking work, pool exhaustion, async behavior, and race-related throughput loss.

## 7. Asset & Payload Optimization (perf-asset-optimization)

**Impact:** LOW
**Description:** Bundles, images, compression, and lazy loading opportunities.
