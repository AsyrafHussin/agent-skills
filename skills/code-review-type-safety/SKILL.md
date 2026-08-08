---
name: code-review-type-safety
description: Type-safety review lens for catching schema mismatches, unsafe coercions, missing annotations, and weak contracts during code review.
license: MIT
metadata:
  author: Agent Skills Team
  version: "1.1.0"
---

# Code Review Type Safety

Targeted type-safety lens for honest contracts, runtime validation, unsafe casts, and generic discipline.

## Review Focus

| Priority | Category | Prefix |
|----------|----------|--------|
| HIGH | Type coverage | `type-type-coverage` |
| CRITICAL | Type correctness | `type-type-correctness` |
| CRITICAL | Type safety | `type-type-safety` |
| MEDIUM | Generic discipline | `type-generic-discipline` |

Look for untyped public contracts, annotations that disagree with behavior, unsafe casts, unvalidated external shapes, weak nullability, and generic machinery that hides rather than proves constraints.

Do not relax an honest contract merely to satisfy current inference. Prove the stricter type or add the missing validation when it still reflects reality.

## Scope

Run this lens when typing/contracts are the dominant concern or a workflow explicitly dispatches it. Do not broaden into a generic review bundle.

Out of scope as primary concerns: security vulnerabilities, retry/timeout behavior, performance-only questions, architecture/readability concerns without a type-contract defect.

## Handoff

When another concern becomes dominant, emit **at most one** focused handoff with the observed `path:line` and why that concern is dominant:

- `code-review-security` for untyped external input at a trust boundary;
- `code-review-architecture` for a leaky or wrongly owned contract;
- `code-review-simplicity` for needless generic machinery or dead constraints;
- `code-review-error-handling` for dishonest failure/result types.

## Evidence Discipline

- Compare declared types with actual producers, consumers, and runtime validation.
- When the diff changes external/untyped input, nullability, coercion, or shape constraints, construct one relevant unexpected value and trace both static proof and runtime behavior. Do not manufacture adversarial typing exercises for unrelated code.
- Prefer explicit domain types and narrow unions/shapes over `mixed`, unchecked casts, or speculative generics.
- If the claimed contract defect depends on unavailable producer/consumer evidence, return `blocked` instead of guessing.

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

- **CRITICAL**: contract likely to break runtime behavior or key invariants.
- **HIGH**: significant unsafe assertion, unvalidated shape, or misleading public type.
- **MEDIUM**: concrete precision/generic-discipline improvement.
- **LOW**: minor type verbosity or style feedback.

## References

- [Pi Ensemble type-safety lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-type-safety/SKILL.md)
- [TypeScript handbook](https://www.typescriptlang.org/docs/)
