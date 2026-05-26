# Code Review Architecture — Full Compiled Reference

Architecture review lens for design quality, coupling, cohesion, module boundaries, and rollback-safe side effects.

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
- "review architecture"
- "architecture review"
- "module boundaries"
- "transaction boundary"
- "separation of concerns"

## Scope Discipline

### In Scope
- Review architectural concerns: design patterns, module structure, coupling, and boundaries
- Analyze separation of concerns and responsibility assignment
- Check abstraction levels, interfaces, and extension points
- Verify transactional safety when durable writes interact with external systems

### Do Not Broaden Into
- ❌ Type errors or type coverage issues
- ❌ Security vulnerabilities as the primary lens
- ❌ Timeout, retry, or exception hygiene details
- ❌ Raw throughput or micro-optimization analysis
- ❌ Pure readability or naming concerns

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

- **CRITICAL**: Architectural violations that can create rollback inconsistency, cascading failures, or long-lived technical debt.
- **HIGH**: Design flaws that materially hinder maintenance, extension, or safe change.
- **MEDIUM**: Structural improvements that would simplify evolution or reduce coupling.
- **LOW**: Minor architectural suggestions with limited risk.

## Metadata Guidance

- **cross_lens_candidate** — true when the finding also suggests another lens should inspect the same location, otherwise false.
- **tradeoff_required** — true when the fix requires meaningful refactoring or behavior trade-offs, otherwise false.

## Adversarial Input Discipline

- Identify one plausible failure scenario for the main architectural change in the diff: rollback during a transaction, lifecycle hook under constraint failure, dependency cycle under load, or a contract change existing callers do not honor.
- For transactional or bulk-persistence diffs, name the transactional entry point, list every persisted type touched inside it, inspect non-post-commit lifecycle hooks on those types, and state whether any hook writes to a non-transactional store.
- If cross-file hook inspection is not performed when required, return BLOCKED instead of APPROVED.

---


## Section 1: Coupling & Cohesion — CRITICAL

Tight coupling, low cohesion, circular dependencies, dependency inversion issues.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to coupling & cohesion.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 2: Separation of Concerns — CRITICAL

Business logic placement, layering, and single-responsibility boundaries.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to separation of concerns.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 3: Abstraction & Interfaces — HIGH

Abstraction levels, interface quality, and leaky boundaries.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to abstraction & interfaces.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 4: Module Boundaries — HIGH

API contracts, visibility, and boundary enforcement.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to module boundaries.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 5: Design Patterns — MEDIUM

Appropriate pattern use, anti-patterns, and consistency.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to design patterns.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 6: Data Flow — MEDIUM

Clear data movement, state handling, and persistence separation.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to data flow.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 7: Extensibility & Maintainability — MEDIUM

Configurability, duplication, and future change safety.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to extensibility & maintainability.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 8: Transaction Boundary Invariants — CRITICAL

Transactional writes versus external side effects and lifecycle-hook safety.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to transaction boundary invariants.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.


---

## Integration Notes

- Part of the six-pass review protocol. Precedence: SECURITY > ERROR_HANDLING > TYPE_SAFETY > PERFORMANCE > ARCHITECTURE > SIMPLICITY.
- Findings merge deterministically across lenses by `(path, line, title)`.
- CRITICAL and HIGH findings belong in `## Must Fix`; MEDIUM and LOW belong in `## Observations` unless repo-specific instructions say otherwise.
