---
name: code-review-performance
description: Performance-focused review lens for identifying inefficiencies, high-cost query shapes, resource waste, and worst-case scalability issues during code review. Triggers on "performance review", "find bottlenecks", "n+1 review", "resource usage", and related review requests.
license: MIT
metadata:
  author: Agent Skills Team
  version: "1.0.0"
---

# Code Review Performance

Performance review lens for algorithmic cost, data access, caching, concurrency, and worst-case resource usage. Contains 7 rules across 7 focused categories for targeted multi-lens review work.

## When to Apply

Reference these guidelines when:
- Running a targeted review for this concern
- Merging findings from a broader multi-lens review
- Stress-testing a risky diff before approval
- Writing or refining repo-owned review instructions for this lens

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Algorithmic Complexity | CRITICAL | `perf-algorithmic-complexity` |
| 2 | Database Performance | CRITICAL | `perf-database-performance` |
| 3 | Network I/O | HIGH | `perf-network-io` |
| 4 | Memory Management | HIGH | `perf-memory-management` |
| 5 | Caching Strategy | MEDIUM | `perf-caching-strategy` |
| 6 | Concurrency | HIGH | `perf-concurrency` |
| 7 | Asset & Payload Optimization | LOW | `perf-asset-optimization` |

## Quick Reference

- `perf-algorithmic-complexity` - Complexity growth, nested loops, and expensive recomputation
- `perf-database-performance` - N+1 queries, indexes, joins, and query-shape efficiency
- `perf-network-io` - Batching, pagination, parallelism, timeouts, and unnecessary round trips
- `perf-memory-management` - Leaks, retention, object copying, and unbounded collections
- `perf-caching-strategy` - Cacheability, invalidation, key design, and cold-path cost
- `perf-concurrency` - Blocking work, pool exhaustion, async behavior, and race-related throughput loss
- `perf-asset-optimization` - Bundles, images, compression, and lazy loading opportunities

## Scope Discipline

Apply this lens when explicitly requested or when a review workflow dispatches it.

### In Scope
- Review performance concerns: efficiency, bottlenecks, and resource use
- Analyze algorithmic complexity and data-access patterns
- Check caching, batching, lazy loading, and concurrency behavior
- Estimate worst-case cost for the modified path

### Out of Scope
- ❌ Primary type-safety analysis
- ❌ Primary security vulnerability review
- ❌ Retry or exception hygiene unless it changes performance behavior
- ❌ Pure architecture or readability concerns

## Output Format

```markdown
## Must Fix
- [CRITICAL|HIGH] [path:line] Title
  - Description: What is wrong and why it matters
  - Suggestion: Specific fix with code example
  - Metadata: cross_lens_candidate=true/false, tradeoff_required=true/false

## Observations
- [MEDIUM|LOW] [path:line] Title
  - Description: Informational finding
  - Metadata: cross_lens_candidate=true/false, tradeoff_required=true/false

## Summary
[One paragraph overall assessment]
```

## Severity Scale

- **CRITICAL**: Regressions likely to cause timeouts, outages, or severe user-facing slowdowns.
- **HIGH**: Measurable inefficiencies or scalability problems with meaningful impact.
- **MEDIUM**: Optimization opportunities with moderate cost savings.
- **LOW**: Minor optimizations or asset-level polish.

## Metadata Guidance Tags

- **cross_lens_candidate** — true when the performance issue also implies architecture, error-handling, or security follow-up, otherwise false.
- **tradeoff_required** — true when the fix introduces complexity or maintenance trade-offs, otherwise false.

## Adversarial Input Discipline

- Construct one concrete high-load or worst-case input for the main code path changed in the diff.
- Estimate the resulting CPU, query, memory, or network cost at that input size.
- If no credible worst-case input can be constructed, return BLOCKED.

## Integration Notes

- Part of the six-pass review protocol. Precedence: SECURITY > ERROR_HANDLING > TYPE_SAFETY > PERFORMANCE > ARCHITECTURE > SIMPLICITY.
- Dedupe merged findings by `(path, line, title)`.
- Lead with the highest-severity must-fix items first.

## References

- [Pi Ensemble performance lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-performance/SKILL.md)
- [Google Web Performance](https://web.dev/explore/fast)
