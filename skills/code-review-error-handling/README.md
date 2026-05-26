# Code Review Error Handling

Resilience-focused review lens for signalling, propagation, retries, cleanup, and observable failure behavior.

## Overview

This skill provides:
- A repo-structured adaptation of the Pi Ensemble review lens
- Focused review guidance for error handling
- A standardized Must Fix / Observations / Summary output contract
- Adversarial-input discipline to avoid shallow approvals

## Categories

### 1. Error Signalling Discipline (Critical)
Ignored return codes, unchecked fallible operations, and silent partial-success paths.
### 2. Exception & Error Hygiene (Critical)
Overbroad catches, swallowed errors, and lost causal context.
### 3. Timeout & Cancellation Discipline (Critical)
Explicit timeouts, cancellation propagation, and bounded waits.
### 4. Retry Semantics (High)
Max attempts, backoff, idempotency, and retryable error classes.
### 5. Partial-Failure Handling (High)
Batch safety, compensation, and caller-visible partial-success contracts.
### 6. Error-Context Observability (Medium)
Logs, trackers, correlation IDs, and actionable context.
### 7. Resource Cleanup on Error Paths (High)
Scoped release patterns for files, locks, sockets, and transactions.
### 8. Defensive-Programming Overreach (Low)
Unnecessary guards and catches that add noise without resilience value.

## Usage

- "error handling review"
- "resilience review"
- "retry logic"
- "Review this diff for error-handling and resilience issues"

## References

- [Pi Ensemble error-handling lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-error-handling/SKILL.md)
- [Release It! patterns](https://pragprog.com/titles/mnee2/release-it-second-edition/)
