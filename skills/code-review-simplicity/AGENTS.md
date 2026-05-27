# Code Review Simplicity — Full Compiled Reference

Simplicity review lens for readability, cognitive load, unnecessary abstraction, duplication, and dead-range logic.

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
- "simplicity review"
- "reduce complexity"
- "readability review"
- "maintainability review"
- "code clarity"

## Scope Discipline

### In Scope
- Review readability, clarity, cognitive load, duplication, and unnecessary abstraction
- Check whether the simplest correct solution is still being used
- Enumerate bound interactions when constants, clamps, floors, or caps change
- Assess whether testing and debugging are made harder by complexity

### Do Not Broaden Into
- ❌ Security or auth flaws as the primary concern
- ❌ Timeout and retry semantics as the main lens
- ❌ Performance-only bottlenecks
- ❌ Architecture or type-system issues unless marked cross-lens

## Cross-Lens Handoff Discipline

Use this as a targeted lens, not a generic review bundle.

- Prove one dominant complexity or maintainability issue first.
- If another concern becomes primary, recommend exactly one next review lens instead of broadening into a vague multi-lens pass.
- Keep the current pass focused on evidence this lens can actually prove.

Smallest likely follow-up lenses:

- `code-review-architecture` when the abstraction drift is structural rather than local clutter
- `code-review-performance` when the complexity mainly creates duplicated expensive work
- `code-review-type-safety` when dead-range logic or confusing control flow comes from dishonest constraints
- `code-review-error-handling` when complexity mainly hides failure paths, cleanup, or observability

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

- **CRITICAL**: Complexity or bound interactions that create dead logic, constant outputs, or code that is effectively not understandable.
- **HIGH**: Material cognitive load or abstraction overhead that increases bug risk.
- **MEDIUM**: Maintainability problems that should be cleaned up soon.
- **LOW**: Minor readability or documentation improvements.

## Metadata Guidance

- **cross_lens_candidate** — true when the simplicity issue likely implies performance, architecture, or other follow-up, otherwise false.
- **tradeoff_required** — true when simplifying requires sacrificing flexibility or another valid trade-off, otherwise false.

## Adversarial Input Discipline

- Always include a `## Bound-range enumeration` section before any verdict other than BLOCKED.
- For each modified variable transformation or changed bound, enumerate category/branch, floor, cap, minimum input to output, mid input to output, maximum input to output, and collapse verdict.
- If the section is missing or incomplete, the review is invalid and must be BLOCKED.
- If any category produces identical outputs across distinct inputs, flag it as a real issue rather than an intentional simplification.

---


## Section 1: Readability & Clarity — HIGH

Naming, magic values, nesting, boolean complexity, and long functions.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to readability & clarity.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 2: Cognitive Load — HIGH

Parameter count, responsibility load, inheritance depth, and mental model cost.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to cognitive load.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 3: Unnecessary Complexity — CRITICAL

Over-engineering, premature abstraction, and patterns without value.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to unnecessary complexity.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 4: Colliding or Redundant Bounds — CRITICAL

Bound-range collapse, dead-range logic, and constant-output paths.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to colliding or redundant bounds.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 5: Code Duplication — MEDIUM

Copy-paste logic and repeated structures that should converge.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to code duplication.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 6: Documentation & Comments — LOW

Missing, outdated, or contradictory explanation.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to documentation & comments.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 7: Naming Conventions — MEDIUM

Non-descriptive, inconsistent, or misleading names.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to naming conventions.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 8: Testing & Debugging Simplicity — LOW

Hidden side effects, difficult debugging, and hard-to-test design.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to testing & debugging simplicity.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.


---

## Integration Notes

- Part of the six-pass review protocol. Precedence: SECURITY > ERROR_HANDLING > TYPE_SAFETY > PERFORMANCE > ARCHITECTURE > SIMPLICITY.
- Findings merge deterministically across lenses by `(path, line, title)`.
- CRITICAL and HIGH findings belong in `## Must Fix`; MEDIUM and LOW belong in `## Observations` unless repo-specific instructions say otherwise.
