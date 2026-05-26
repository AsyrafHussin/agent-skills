# Code Review Performance — Full Compiled Reference

Performance review lens for algorithmic cost, data access, caching, concurrency, and worst-case resource usage.

**Version:** 1.0.0 | **Rules:** 7 | **License:** MIT

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
- "performance review"
- "find bottlenecks"
- "n+1 review"
- "resource usage"
- "scalability review"

## Scope Discipline

### In Scope
- Review performance concerns: efficiency, bottlenecks, and resource use
- Analyze algorithmic complexity and data-access patterns
- Check caching, batching, lazy loading, and concurrency behavior
- Estimate worst-case cost for the modified path

### Do Not Broaden Into
- ❌ Primary type-safety analysis
- ❌ Primary security vulnerability review
- ❌ Retry or exception hygiene unless it changes performance behavior
- ❌ Pure architecture or readability concerns

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

- **CRITICAL**: Regressions likely to cause timeouts, outages, or severe user-facing slowdowns.
- **HIGH**: Measurable inefficiencies or scalability problems with meaningful impact.
- **MEDIUM**: Optimization opportunities with moderate cost savings.
- **LOW**: Minor optimizations or asset-level polish.

## Metadata Guidance

- **cross_lens_candidate** — true when the performance issue also implies architecture, error-handling, or security follow-up, otherwise false.
- **tradeoff_required** — true when the fix introduces complexity or maintenance trade-offs, otherwise false.

## Adversarial Input Discipline

- Construct one concrete high-load or worst-case input for the main code path changed in the diff.
- Estimate the resulting CPU, query, memory, or network cost at that input size.
- If no credible worst-case input can be constructed, return BLOCKED.

---


## Section 1: Algorithmic Complexity — CRITICAL

Complexity growth, nested loops, and expensive recomputation.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to algorithmic complexity.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 2: Database Performance — CRITICAL

N+1 queries, indexes, joins, and query-shape efficiency.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to database performance.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 3: Network I/O — HIGH

Batching, pagination, parallelism, timeouts, and unnecessary round trips.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to network i/o.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 4: Memory Management — HIGH

Leaks, retention, object copying, and unbounded collections.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to memory management.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 5: Caching Strategy — MEDIUM

Cacheability, invalidation, key design, and cold-path cost.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to caching strategy.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 6: Concurrency — HIGH

Blocking work, pool exhaustion, async behavior, and race-related throughput loss.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to concurrency.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 7: Asset & Payload Optimization — LOW

Bundles, images, compression, and lazy loading opportunities.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to asset & payload optimization.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.


---

## Integration Notes

- Part of the six-pass review protocol. Precedence: SECURITY > ERROR_HANDLING > TYPE_SAFETY > PERFORMANCE > ARCHITECTURE > SIMPLICITY.
- Findings merge deterministically across lenses by `(path, line, title)`.
- CRITICAL and HIGH findings belong in `## Must Fix`; MEDIUM and LOW belong in `## Observations` unless repo-specific instructions say otherwise.
