# Code Review Type Safety — Full Compiled Reference

Type-safety review lens for public API typing, runtime validation, unsafe casts, and generic discipline.

**Version:** 1.0.0 | **Rules:** 4 | **License:** MIT

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
- "type safety review"
- "review typings"
- "schema mismatch"
- "unsafe cast"
- "type coverage"

## Scope Discipline

### In Scope
- Review type-related concerns: type coverage, schema alignment, casts, and generic constraints
- Check public APIs and risky internal paths for precise, honest types
- Verify runtime validation where untyped input crosses a trust boundary
- Assess type guards and annotations against actual behavior
- Flag changes that relax a previously honest contract only to satisfy current tooling or inference; recommend restoring proof or validation when the stricter contract is still correct.

### Do Not Broaden Into
- ❌ Security issues as the primary lens
- ❌ Retry, timeout, or other resilience behavior
- ❌ Performance-only questions
- ❌ Architecture or readability concerns unless clearly cross-lens

## Cross-Lens Handoff Discipline

Use this as a targeted lens, not a generic review bundle.

- Prove one dominant contract or typing issue first.
- If another concern becomes primary, recommend exactly one next review lens instead of broadening into a vague multi-lens pass.
- Keep the current pass focused on evidence this lens can actually prove.

Smallest likely follow-up lenses:

- `code-review-security` when the real issue is untyped external input at a trust boundary
- `code-review-architecture` when the contract mismatch is really a leaky boundary or wrong abstraction
- `code-review-simplicity` when generic machinery or dead constraints are the true problem
- `code-review-error-handling` when dishonest types mainly hide failure contracts

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

- **CRITICAL**: Type issues likely to break compilation, runtime behavior, or key contracts.
- **HIGH**: Significant typing gaps, unsafe assertions, or unvalidated external shapes.
- **MEDIUM**: Helpful precision or generic-discipline improvements.
- **LOW**: Minor verbosity or style-level type feedback.

## Metadata Guidance

- **cross_lens_candidate** — true when the type issue also indicates security or architectural risk, otherwise false.
- **tradeoff_required** — true when stricter types would force an API or ergonomics trade-off, otherwise false.

## Adversarial Input Discipline

- Construct one concrete input with unexpected shape, type, or nullability for the main changed path.
- State what the type system proves and what runtime validation does with that input.
- If no adversarial typed input can be constructed, return BLOCKED.

---


## Section 1: Type Coverage — HIGH

Typed public APIs, critical internals, and avoiding implicit any gaps.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to type coverage.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 2: Type Correctness — CRITICAL

Annotations, assertions, and guards that match real behavior.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to type correctness.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 3: Type Safety — CRITICAL

Unsafe casts, unknown data, discriminated unions, and runtime checks.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to type safety.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.

---


## Section 4: Generic Discipline — MEDIUM

Constraints, parameter bounds, and avoiding generic overuse.

### What to Review
- Apply this rule when the diff touches logic, interfaces, data flow, or behavior related to generic discipline.
- Prefer concrete evidence from the changed code and any directly coupled files you must inspect to validate the finding.
- Report a concrete fix suggestion instead of abstract criticism.


---

## Integration Notes

- Part of the six-pass review protocol. Precedence: SECURITY > ERROR_HANDLING > TYPE_SAFETY > PERFORMANCE > ARCHITECTURE > SIMPLICITY.
- Findings merge deterministically across lenses by `(path, line, title)`.
- CRITICAL and HIGH findings belong in `## Must Fix`; MEDIUM and LOW belong in `## Observations` unless repo-specific instructions say otherwise.
