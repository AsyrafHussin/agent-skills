---
name: code-review-type-safety
description: Type-safety review lens for catching schema mismatches, unsafe coercions, missing annotations, and weak contracts during code review. Triggers on "type safety review", "review typings", "schema mismatch", "unsafe cast", and related review requests.
license: MIT
metadata:
  author: Agent Skills Team
  version: "1.0.0"
---

# Code Review Type Safety

Type-safety review lens for public API typing, runtime validation, unsafe casts, and generic discipline. Contains 4 rules across 4 focused categories for targeted multi-lens review work.

## When to Apply

Reference these guidelines when:
- Running a targeted review for this concern
- Merging findings from a broader multi-lens review
- Stress-testing a risky diff before approval
- Writing or refining repo-owned review instructions for this lens

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Type Coverage | HIGH | `type-type-coverage` |
| 2 | Type Correctness | CRITICAL | `type-type-correctness` |
| 3 | Type Safety | CRITICAL | `type-type-safety` |
| 4 | Generic Discipline | MEDIUM | `type-generic-discipline` |

## Quick Reference

- `type-type-coverage` - Typed public APIs, critical internals, and avoiding implicit any gaps
- `type-type-correctness` - Annotations, assertions, and guards that match real behavior
- `type-type-safety` - Unsafe casts, unknown data, discriminated unions, and runtime checks
- `type-generic-discipline` - Constraints, parameter bounds, and avoiding generic overuse

## Scope Discipline

Apply this lens when explicitly requested or when a review workflow dispatches it.

### In Scope
- Review type-related concerns: type coverage, schema alignment, casts, and generic constraints
- Check public APIs and risky internal paths for precise, honest types
- Verify runtime validation where untyped input crosses a trust boundary
- Assess type guards and annotations against actual behavior

### Out of Scope
- ❌ Security issues as the primary lens
- ❌ Retry, timeout, or other resilience behavior
- ❌ Performance-only questions
- ❌ Architecture or readability concerns unless clearly cross-lens

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

- **CRITICAL**: Type issues likely to break compilation, runtime behavior, or key contracts.
- **HIGH**: Significant typing gaps, unsafe assertions, or unvalidated external shapes.
- **MEDIUM**: Helpful precision or generic-discipline improvements.
- **LOW**: Minor verbosity or style-level type feedback.

## Metadata Guidance Tags

- **cross_lens_candidate** — true when the type issue also indicates security or architectural risk, otherwise false.
- **tradeoff_required** — true when stricter types would force an API or ergonomics trade-off, otherwise false.

## Adversarial Input Discipline

- Construct one concrete input with unexpected shape, type, or nullability for the main changed path.
- State what the type system proves and what runtime validation does with that input.
- If no adversarial typed input can be constructed, return BLOCKED.

## Integration Notes

- Part of the six-pass review protocol. Precedence: SECURITY > ERROR_HANDLING > TYPE_SAFETY > PERFORMANCE > ARCHITECTURE > SIMPLICITY.
- Dedupe merged findings by `(path, line, title)`.
- Lead with the highest-severity must-fix items first.

## References

- [Pi Ensemble type-safety lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-type-safety/SKILL.md)
- [TypeScript handbook](https://www.typescriptlang.org/docs/)
