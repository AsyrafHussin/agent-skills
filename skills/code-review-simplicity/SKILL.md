---
name: code-review-simplicity
description: Simplicity-focused review lens for identifying unnecessary complexity, dead logic, unclear naming, duplicated behavior, and maintainability drag during code review.
license: MIT
metadata:
  author: Agent Skills Team
  version: "1.1.0"
---

# Code Review Simplicity

Targeted simplicity lens for readability, cognitive load, unnecessary abstraction, duplication, and dead logic.

## Review Focus

| Priority | Category | Prefix |
|----------|----------|--------|
| HIGH | Readability & clarity | `simp-readability-clarity` |
| HIGH | Cognitive load | `simp-cognitive-load` |
| CRITICAL | Unnecessary complexity | `simp-unnecessary-complexity` |
| CRITICAL | Colliding/redundant bounds | `simp-colliding-bounds` |
| MEDIUM | Duplication | `simp-code-duplication` |
| LOW | Documentation/comments | `simp-documentation-comments` |
| MEDIUM | Naming | `simp-naming-conventions` |
| LOW | Testing/debugging simplicity | `simp-testing-debugging` |

Prefer, in order: delete code, reuse an existing owner, use the language/platform, inline a needless abstraction, then add new structure only when the verified behavior requires it.

Clean-code checks are concrete: intention-revealing names, small cohesive units with one reason to change, narrow interfaces, shallow control flow, and no helper/manager/configuration dumping ground created merely to move complexity elsewhere.

## Scope

Run this lens when complexity or maintainability is the dominant concern or a workflow explicitly dispatches it. Do not broaden into a generic review bundle.

Out of scope as primary concerns: security, retry semantics, performance-only bottlenecks, architecture/type defects that merely happen to look complex.

Do not call required validation, contextual exceptions, security checks, or focused regression tests "bloat".

## Handoff

When another concern becomes dominant, emit **at most one** focused handoff with the observed `path:line` and why that concern is dominant:

- `code-review-architecture` for structural ownership/abstraction defects;
- `code-review-performance` for duplicated expensive work;
- `code-review-type-safety` for dishonest constraints or dead typed branches;
- `code-review-error-handling` for hidden failure paths or cleanup.

## Evidence Discipline

- Tie each finding to real source and callers; do not infer dead code from the diff alone.
- Rank by maintenance surface removed, not cleverness.
- When constants, clamps, floors, caps, or range transformations changed, enumerate the relevant boundary cases and check for collapsed/dead ranges. Otherwise do **not** manufacture a bound-range exercise.
- Prefer a local deletion/simplification over a new helper, interface, manager, strategy, or configuration point with no demonstrated second use.
- If removal safety depends on unavailable callers or behavior, return `blocked` instead of guessing.

## Terminal Contract

```text
STATUS: findings
<path>:<line>: <CRITICAL|HIGH|MEDIUM|LOW> <problem>. <smaller replacement>.
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

- **CRITICAL**: dead/collapsed logic or complexity that obscures correctness.
- **HIGH**: material cognitive or abstraction overhead that increases bug risk.
- **MEDIUM**: concrete maintainability drag.
- **LOW**: minor readability/documentation issue.

## References

- [Pi Ensemble simplicity lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-simplicity/SKILL.md)
- [Refactoring Guru code smells](https://refactoring.guru/refactoring/smells)
