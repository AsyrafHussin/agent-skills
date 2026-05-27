---
name: code-review-error-handling
description: Error-handling and resilience review lens for catching swallowed failures, unbounded I/O, missing timeouts, retry hazards, and partial-failure bugs during code review. Triggers on "error handling review", "resilience review", "retry logic", "timeout discipline", and related review requests.
license: MIT
metadata:
  author: Agent Skills Team
  version: "1.0.0"
---

# Code Review Error Handling

Resilience-focused review lens for signalling, propagation, retries, cleanup, and observable failure behavior. Contains 8 rules across 8 focused categories for targeted multi-lens review work.

## When to Apply

Reference these guidelines when:
- Running a targeted review for this concern
- Merging findings from a broader multi-lens review
- Stress-testing a risky diff before approval
- Writing or refining repo-owned review instructions for this lens

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Error Signalling Discipline | CRITICAL | `err-signalling-discipline` |
| 2 | Exception & Error Hygiene | CRITICAL | `err-exception-hygiene` |
| 3 | Timeout & Cancellation Discipline | CRITICAL | `err-timeout-cancellation` |
| 4 | Retry Semantics | HIGH | `err-retry-semantics` |
| 5 | Partial-Failure Handling | HIGH | `err-partial-failure` |
| 6 | Error-Context Observability | MEDIUM | `err-observability` |
| 7 | Resource Cleanup on Error Paths | HIGH | `err-resource-cleanup` |
| 8 | Defensive-Programming Overreach | LOW | `err-defensive-overreach` |

## Quick Reference

- `err-signalling-discipline` - Ignored return codes, unchecked fallible operations, and silent partial-success paths
- `err-exception-hygiene` - Overbroad catches, swallowed errors, and lost causal context
- `err-timeout-cancellation` - Explicit timeouts, cancellation propagation, and bounded waits
- `err-retry-semantics` - Max attempts, backoff, idempotency, and retryable error classes
- `err-partial-failure` - Batch safety, compensation, and caller-visible partial-success contracts
- `err-observability` - Logs, trackers, correlation IDs, and actionable context
- `err-resource-cleanup` - Scoped release patterns for files, locks, sockets, and transactions
- `err-defensive-overreach` - Unnecessary guards and catches that add noise without resilience value
- Treat catches as recovery, translation, or logging boundaries; if a catch block does none of those, question whether it should exist.

## Scope Discipline

Apply this lens when explicitly requested or when a review workflow dispatches it.

### In Scope
- Review failure signalling, propagation, containment, and recovery behavior
- Check timeout, cancellation, retry, and resource-cleanup discipline on fallible operations
- Verify that errors remain observable with enough context for diagnosis
- Assess partial-failure handling for batch and multi-step flows

### Out of Scope
- ❌ Primary security vulnerability analysis
- ❌ Type-system coverage or generic design
- ❌ Raw performance tuning outside resilience concerns
- ❌ Pure architecture or readability refactors

## Cross-Lens Handoff Discipline

Use this as a targeted lens, not a generic review bundle.

- Prove one dominant resilience or failure-path issue first.
- If another concern becomes primary, recommend exactly one next review lens instead of broadening into a vague multi-lens pass.
- Keep the current pass focused on evidence this lens can actually prove.

Smallest likely follow-up lenses:

- `code-review-security` when the failure path leaks secrets, bypasses controls, or weakens authorization
- `code-review-performance` when retries, timeouts, backpressure, or blocking behavior are the dominant risk
- `code-review-architecture` when the core issue is transaction or boundary design rather than local cleanup logic
- `code-review-type-safety` when the main defect is a dishonest error/result contract

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

- **CRITICAL**: Patterns that can cause silent corruption, cascading failures, leaked resources, or unrecoverable state.
- **HIGH**: Likely production incidents or lost observability, including missing timeouts or unsafe retries.
- **MEDIUM**: Reduced diagnosability or robustness under stress.
- **LOW**: Minor hygiene or over-defensive code that adds maintenance cost.

## Metadata Guidance Tags

- **cross_lens_candidate** — true when the finding also affects security, architecture, or performance, otherwise false.
- **tradeoff_required** — true when the fix changes observable behavior or caller contracts, otherwise false.

## Adversarial Input Discipline

- Pick one external dependency or fallible operation the diff introduces or modifies.
- Trace four scenarios: timeout, transient failure, permanent failure, and eventual success after earlier silent failures.
- For each scenario, state what is logged, retried, persisted, returned to the caller, and whether any resource leaks remain.
- If the four scenarios cannot be traced, return BLOCKED.

## Integration Notes

- Part of the six-pass review protocol. Precedence: SECURITY > ERROR_HANDLING > TYPE_SAFETY > PERFORMANCE > ARCHITECTURE > SIMPLICITY.
- Dedupe merged findings by `(path, line, title)`.
- Lead with the highest-severity must-fix items first.

## References

- [Pi Ensemble error-handling lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-error-handling/SKILL.md)
- [Release It! patterns](https://pragprog.com/titles/mnee2/release-it-second-edition/)
