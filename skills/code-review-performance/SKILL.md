---
name: code-review-performance
description: Performance-focused review lens for identifying inefficiencies, high-cost query shapes, resource waste, and worst-case scalability issues during code review.
license: MIT
metadata:
  author: Agent Skills Team
  version: "1.1.0"
---

# Code Review Performance

Targeted performance lens for algorithmic cost, data access, caching, concurrency, and worst-case resource usage.

## Review Focus

| Priority | Category | Prefix |
|----------|----------|--------|
| CRITICAL | Algorithmic complexity | `perf-algorithmic-complexity` |
| CRITICAL | Database performance | `perf-database-performance` |
| HIGH | Network I/O | `perf-network-io` |
| HIGH | Memory management | `perf-memory-management` |
| MEDIUM | Caching strategy | `perf-caching-strategy` |
| HIGH | Concurrency | `perf-concurrency` |
| LOW | Asset & payload optimization | `perf-asset-optimization` |

Look for complexity growth, N+1 queries, missing indexes, redundant round trips, unbounded collections, bad cache invalidation, blocking work, pool pressure, and user-visible payload waste.

## Scope

Run this lens when cost, latency, throughput, or scalability is the dominant concern or a workflow explicitly dispatches it. Do not broaden into a generic review bundle.

Out of scope as primary concerns: type safety, security, retry hygiene without performance impact, architecture-only refactors, readability-only feedback.

## Handoff

When another concern becomes dominant, emit **at most one** focused handoff with the observed `path:line` and why that concern is dominant:

- `code-review-architecture` for ownership/layering defects causing the cost;
- `code-review-error-handling` for retries, timeout, or recovery behavior;
- `code-review-simplicity` for duplicated or over-abstracted work;
- `code-review-security` for adversarial resource exhaustion.

## Evidence Discipline

- Tie each finding to an observed cost path, query shape, allocation pattern, network call, or concurrency boundary.
- When the changed path can amplify work with input size, construct one credible worst-case input and estimate the relevant CPU/query/memory/network growth. Do not block unrelated changes merely because no synthetic worst case exists.
- Prefer removing work over adding caches or concurrency machinery when deletion solves the measured problem.
- If the claimed regression depends on unavailable runtime/query evidence, return `blocked` instead of guessing.

## Terminal Contract

```text
STATUS: findings
<path>:<line>: <CRITICAL|HIGH|MEDIUM|LOW> <problem>. <concrete fix>.
HANDOFF: <code-review-* lens> <path>:<line> <why this concern is dominant>   # optional, at most one
```

```text
STATUS: clean
```

```text
STATUS: blocked
UNKNOWN: <exact missing evidence>.
```

`STATUS` is this lens' judgment only. The caller owns merge, dedupe, precedence, approval, persistence, and workflow progression.

## Severity

- **CRITICAL**: likely timeout, outage, or severe user-facing slowdown.
- **HIGH**: measurable inefficiency or scalability problem with meaningful impact.
- **MEDIUM**: clear optimization with moderate savings.
- **LOW**: minor optimization or payload polish.

## References

- [Pi Ensemble performance lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-performance/SKILL.md)
- [Google Web Performance](https://web.dev/explore/fast)
