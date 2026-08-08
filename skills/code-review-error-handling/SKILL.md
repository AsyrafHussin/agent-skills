---
name: code-review-error-handling
description: Error-handling and resilience review lens for catching swallowed failures, unbounded I/O, missing timeouts, retry hazards, and partial-failure bugs during code review.
license: MIT
metadata:
  author: Agent Skills Team
  version: "1.1.0"
---

# Code Review Error Handling

Targeted resilience lens for signalling, propagation, retries, cleanup, and observable failure behavior.

## Review Focus

| Priority | Category | Prefix |
|----------|----------|--------|
| CRITICAL | Error signalling | `err-signalling-discipline` |
| CRITICAL | Exception hygiene | `err-exception-hygiene` |
| CRITICAL | Timeout & cancellation | `err-timeout-cancellation` |
| HIGH | Retry semantics | `err-retry-semantics` |
| HIGH | Partial failure | `err-partial-failure` |
| MEDIUM | Observability | `err-observability` |
| HIGH | Resource cleanup | `err-resource-cleanup` |
| LOW | Defensive overreach | `err-defensive-overreach` |

Look for ignored results, swallowed exceptions, missing causal context, unbounded waits, unsafe retries, caller-invisible partial success, leaked resources, and guards that add noise without recovery value.

## Scope

Run this lens when resilience or failure behavior is the dominant concern or a workflow explicitly dispatches it. Do not broaden into a generic review bundle.

Out of scope as primary concerns: security vulnerabilities, type-system design, raw performance tuning, architecture-only refactors, readability-only feedback.

## Handoff

When another concern becomes dominant, emit **at most one** focused handoff with the observed `path:line` and why that concern is dominant:

- `code-review-security` for leaked secrets or weakened controls;
- `code-review-performance` for retry/backpressure/blocking cost;
- `code-review-architecture` for structural transaction/boundary defects;
- `code-review-type-safety` for dishonest error/result contracts.

## Evidence Discipline

- Trace the actual failure path before proposing recovery behavior.
- When the diff changes an external or fallible operation, test the relevant timeout, transient failure, permanent failure, and eventual-success paths. Do not manufacture scenarios for code that has no such operation.
- State what is logged, retried, persisted, returned, and cleaned up when those behaviors matter to the finding.
- If required failure-path evidence cannot be inspected, return `blocked` instead of guessing.
- Prefer one clear failure contract over defensive layers that merely hide uncertainty.

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

- **CRITICAL**: silent corruption, cascading failure, unrecoverable state, or severe resource leakage.
- **HIGH**: likely incident, lost observability, missing timeout, or unsafe retry.
- **MEDIUM**: reduced diagnosability or robustness.
- **LOW**: minor hygiene or needless defensive complexity.

## References

- [Pi Ensemble error-handling lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-error-handling/SKILL.md)
- [Release It! patterns](https://pragprog.com/titles/mnee2/release-it-second-edition/)
