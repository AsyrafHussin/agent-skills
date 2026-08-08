---
name: code-review-architecture
description: Architecture-focused review lens for assessing coupling, module boundaries, abstraction quality, and rollback-safe design during code review. Use when reviewing design quality, separation of concerns, transaction boundaries, or structural maintainability.
license: MIT
metadata:
  author: Agent Skills Team
  version: "1.1.0"
---

# Code Review Architecture

Targeted architecture lens for design quality, coupling, cohesion, boundaries, and rollback-safe side effects.

## Review Focus

| Priority | Category | Prefix |
|----------|----------|--------|
| CRITICAL | Coupling & cohesion | `arch-coupling-cohesion` |
| CRITICAL | Separation of concerns | `arch-separation-concerns` |
| HIGH | Abstraction & interfaces | `arch-abstraction-interfaces` |
| HIGH | Module boundaries | `arch-module-boundaries` |
| MEDIUM | Design patterns | `arch-design-patterns` |
| MEDIUM | Data flow | `arch-data-flow` |
| MEDIUM | Extensibility & maintainability | `arch-extensibility-maintainability` |
| CRITICAL | Transaction boundaries | `arch-transaction-boundaries` |

Look for circular dependencies, misplaced business logic, leaky interfaces, unnecessary indirection, boundary violations, unclear data ownership, and durable writes mixed unsafely with external side effects.

## Scope

Run this lens when architecture is the dominant concern or a workflow explicitly dispatches it. Do not broaden into a generic review bundle.

Out of scope as primary concerns: type coverage, security vulnerabilities, timeout/retry hygiene, micro-optimization, naming-only feedback.

## Handoff

Stay on architecture evidence. When another concern becomes dominant, emit **at most one** focused handoff with the observed `path:line` and why that concern is dominant:

- `code-review-performance` for cost/query/resource growth;
- `code-review-error-handling` for local rollback, cleanup, or partial failure;
- `code-review-security` for authz, isolation, or trust boundaries;
- `code-review-simplicity` for over-abstraction without an architectural defect.

A handoff is a recommendation, not workflow approval and not permission to run every lens.

## Evidence Discipline

- Tie each finding to real source and caller/boundary evidence.
- For transactional or bulk-persistence changes, inspect the transaction entry point, persisted types, and relevant lifecycle hooks.
- If a claimed architectural defect depends on unavailable cross-file evidence, return `blocked` instead of guessing.
- Prefer deletion and direct ownership over a new abstraction when both solve the verified problem.

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

- **CRITICAL**: rollback inconsistency, cascading failure, or a broken architectural boundary.
- **HIGH**: design flaw that materially hinders safe change.
- **MEDIUM**: structural improvement with clear maintenance value.
- **LOW**: minor architectural suggestion.

## References

- [Pi Ensemble architecture lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-architecture/SKILL.md)
- [Martin Fowler on refactoring](https://refactoring.com/)
