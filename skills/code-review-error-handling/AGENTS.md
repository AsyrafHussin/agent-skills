# Code Review Error Handling — Full Compiled Reference

Resilience-focused review lens for signalling, propagation, retries, cleanup, and observable failure behavior.

**Version:** 1.0.0 | **Rules:** 8 | **License:** MIT

---

## Operational Contract

When applying this skill, agents must:
- Treat this skill as repo-owned guidance and defer to repository or task-specific instructions when they conflict.
- Limit work to the smallest relevant file and rule set for the current request.
- Stop and ask when the scope, validation command, or required context is missing or contradictory.
- Prefer machine-readable evidence first, then summarize files reviewed, commands run, failures, and unresolved risks.

## Validation & Evidence

- Run the repository's existing validation commands in documented order when code changes are requested.
- If the repository does not define validation for the task, say so instead of inventing one.
- When the lens requires cross-file inspection, name the extra files reviewed.

## Trigger Phrases

The skill activates on:
- "error handling review"
- "resilience review"
- "retry logic"
- "timeout discipline"
- "partial failure"

## Scope Discipline

### In Scope
- Review failure signalling, propagation, containment, and recovery behavior
- Check timeout, cancellation, retry, and resource-cleanup discipline on fallible operations
- Verify that errors remain observable with enough context for diagnosis
- Assess partial-failure handling for batch and multi-step flows

### Do Not Broaden Into
- ❌ Primary security vulnerability analysis
- ❌ Type-system coverage or generic design
- ❌ Raw performance tuning outside resilience concerns
- ❌ Pure architecture or readability refactors

## Output Format

Use this exact structure:

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

## Metadata Guidance

- **cross_lens_candidate** — true when the finding also affects security, architecture, or performance, otherwise false.
- **tradeoff_required** — true when the fix changes observable behavior or caller contracts, otherwise false.

## Adversarial Input Discipline

- Pick one external dependency or fallible operation the diff introduces or modifies.
- Trace four scenarios: timeout, transient failure, permanent failure, and eventual success after earlier silent failures.
- For each scenario, state what is logged, retried, persisted, returned to the caller, and whether any resource leaks remain.
- If the four scenarios cannot be traced, return BLOCKED.

---


## Section 1: Error Signalling Discipline — CRITICAL

Ignored return codes, unchecked fallible operations, and silent partial-success paths.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to error signalling discipline.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 2: Exception & Error Hygiene — CRITICAL

Overbroad catches, swallowed errors, and lost causal context.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to exception & error hygiene.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 3: Timeout & Cancellation Discipline — CRITICAL

Explicit timeouts, cancellation propagation, and bounded waits.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to timeout & cancellation discipline.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 4: Retry Semantics — HIGH

Max attempts, backoff, idempotency, and retryable error classes.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to retry semantics.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 5: Partial-Failure Handling — HIGH

Batch safety, compensation, and caller-visible partial-success contracts.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to partial-failure handling.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 6: Error-Context Observability — MEDIUM

Logs, trackers, correlation IDs, and actionable context.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to error-context observability.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 7: Resource Cleanup on Error Paths — HIGH

Scoped release patterns for files, locks, sockets, and transactions.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to resource cleanup on error paths.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 8: Defensive-Programming Overreach — LOW

Unnecessary guards and catches that add noise without resilience value.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to defensive-programming overreach.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.


---

## Integration Notes

- Part of the six-pass review protocol. Precedence: SECURITY > ERROR_HANDLING > TYPE_SAFETY > PERFORMANCE > ARCHITECTURE > SIMPLICITY.
- Findings merge deterministically across lenses by `(path, line, title)`.
- CRITICAL and HIGH findings belong in `## Must Fix`; MEDIUM and LOW belong in `## Observations` unless repo-specific instructions say otherwise.
