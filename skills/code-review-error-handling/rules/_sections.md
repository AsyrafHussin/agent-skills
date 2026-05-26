# Sections

This file defines the section ordering, severity, and summary used by this review lens.

---

## 1. Error Signalling Discipline (err-signalling-discipline)

**Impact:** CRITICAL
**Description:** Ignored return codes, unchecked fallible operations, and silent partial-success paths.

## 2. Exception & Error Hygiene (err-exception-hygiene)

**Impact:** CRITICAL
**Description:** Overbroad catches, swallowed errors, and lost causal context.

## 3. Timeout & Cancellation Discipline (err-timeout-cancellation)

**Impact:** CRITICAL
**Description:** Explicit timeouts, cancellation propagation, and bounded waits.

## 4. Retry Semantics (err-retry-semantics)

**Impact:** HIGH
**Description:** Max attempts, backoff, idempotency, and retryable error classes.

## 5. Partial-Failure Handling (err-partial-failure)

**Impact:** HIGH
**Description:** Batch safety, compensation, and caller-visible partial-success contracts.

## 6. Error-Context Observability (err-observability)

**Impact:** MEDIUM
**Description:** Logs, trackers, correlation IDs, and actionable context.

## 7. Resource Cleanup on Error Paths (err-resource-cleanup)

**Impact:** HIGH
**Description:** Scoped release patterns for files, locks, sockets, and transactions.

## 8. Defensive-Programming Overreach (err-defensive-overreach)

**Impact:** LOW
**Description:** Unnecessary guards and catches that add noise without resilience value.
