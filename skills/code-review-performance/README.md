# Code Review Performance

Performance review lens for algorithmic cost, data access, caching, concurrency, and worst-case resource usage.

## Overview

This skill provides:
- A repo-structured adaptation of the Pi Ensemble review lens
- Focused review guidance for performance
- A standardized Must Fix / Observations / Summary output contract
- Adversarial-input discipline to avoid shallow approvals

## Categories

### 1. Algorithmic Complexity (Critical)
Complexity growth, nested loops, and expensive recomputation.
### 2. Database Performance (Critical)
N+1 queries, indexes, joins, and query-shape efficiency.
### 3. Network I/O (High)
Batching, pagination, parallelism, timeouts, and unnecessary round trips.
### 4. Memory Management (High)
Leaks, retention, object copying, and unbounded collections.
### 5. Caching Strategy (Medium)
Cacheability, invalidation, key design, and cold-path cost.
### 6. Concurrency (High)
Blocking work, pool exhaustion, async behavior, and race-related throughput loss.
### 7. Asset & Payload Optimization (Low)
Bundles, images, compression, and lazy loading opportunities.

## Usage

- "performance review"
- "find bottlenecks"
- "n+1 review"
- "Review this diff for performance regressions"

## References

- [Pi Ensemble performance lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-performance/SKILL.md)
- [Google Web Performance](https://web.dev/explore/fast)
