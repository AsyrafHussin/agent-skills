---
name: code-review-simplicity
description: Simplicity-focused review lens for identifying unnecessary complexity, dead logic, unclear naming, duplicated behavior, and maintainability drag during code review. Triggers on "simplicity review", "reduce complexity", "readability review", "maintainability review", and related review requests.
license: MIT
metadata:
  author: Agent Skills Team
  version: "1.0.0"
---

# Code Review Simplicity

Simplicity review lens for readability, cognitive load, unnecessary abstraction, duplication, and dead-range logic. Contains 8 rules across 8 focused categories for targeted multi-lens review work.

## When to Apply

Reference these guidelines when:
- Running a targeted review for this concern
- Merging findings from a broader multi-lens review
- Stress-testing a risky diff before approval
- Writing or refining repo-owned review instructions for this lens

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Readability & Clarity | HIGH | `simp-readability-clarity` |
| 2 | Cognitive Load | HIGH | `simp-cognitive-load` |
| 3 | Unnecessary Complexity | CRITICAL | `simp-unnecessary-complexity` |
| 4 | Colliding or Redundant Bounds | CRITICAL | `simp-colliding-bounds` |
| 5 | Code Duplication | MEDIUM | `simp-code-duplication` |
| 6 | Documentation & Comments | LOW | `simp-documentation-comments` |
| 7 | Naming Conventions | MEDIUM | `simp-naming-conventions` |
| 8 | Testing & Debugging Simplicity | LOW | `simp-testing-debugging` |

## Quick Reference

- `simp-readability-clarity` - Naming, magic values, nesting, boolean complexity, and long functions
- `simp-cognitive-load` - Parameter count, responsibility load, inheritance depth, and mental model cost
- `simp-unnecessary-complexity` - Over-engineering, premature abstraction, and patterns without value
- `simp-colliding-bounds` - Bound-range collapse, dead-range logic, and constant-output paths
- `simp-code-duplication` - Copy-paste logic and repeated structures that should converge
- `simp-documentation-comments` - Missing, outdated, or contradictory explanation
- `simp-naming-conventions` - Non-descriptive, inconsistent, or misleading names
- `simp-testing-debugging` - Hidden side effects, difficult debugging, and hard-to-test design

## Scope Discipline

Apply this lens when explicitly requested or when a review workflow dispatches it.

### In Scope
- Review readability, clarity, cognitive load, duplication, and unnecessary abstraction
- Check whether the simplest correct solution is still being used
- Enumerate bound interactions when constants, clamps, floors, or caps change
- Assess whether testing and debugging are made harder by complexity

### Out of Scope
- ❌ Security or auth flaws as the primary concern
- ❌ Timeout and retry semantics as the main lens
- ❌ Performance-only bottlenecks
- ❌ Architecture or type-system issues unless marked cross-lens

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

- **CRITICAL**: Complexity or bound interactions that create dead logic, constant outputs, or code that is effectively not understandable.
- **HIGH**: Material cognitive load or abstraction overhead that increases bug risk.
- **MEDIUM**: Maintainability problems that should be cleaned up soon.
- **LOW**: Minor readability or documentation improvements.

## Metadata Guidance Tags

- **cross_lens_candidate** — true when the simplicity issue likely implies performance, architecture, or other follow-up, otherwise false.
- **tradeoff_required** — true when simplifying requires sacrificing flexibility or another valid trade-off, otherwise false.

## Adversarial Input Discipline

- Always include a `## Bound-range enumeration` section before any verdict other than BLOCKED.
- For each modified variable transformation or changed bound, enumerate category/branch, floor, cap, minimum input to output, mid input to output, maximum input to output, and collapse verdict.
- If the section is missing or incomplete, the review is invalid and must be BLOCKED.
- If any category produces identical outputs across distinct inputs, flag it as a real issue rather than an intentional simplification.

## Integration Notes

- Part of the six-pass review protocol. Precedence: SECURITY > ERROR_HANDLING > TYPE_SAFETY > PERFORMANCE > ARCHITECTURE > SIMPLICITY.
- Dedupe merged findings by `(path, line, title)`.
- Lead with the highest-severity must-fix items first.

## References

- [Pi Ensemble simplicity lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-simplicity/SKILL.md)
- [Refactoring Guru code smells](https://refactoring.guru/refactoring/smells)
