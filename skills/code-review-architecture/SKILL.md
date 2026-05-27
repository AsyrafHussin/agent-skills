---
name: code-review-architecture
description: Architecture-focused review lens for assessing coupling, module boundaries, abstraction quality, and rollback-safe design during code review. Use when reviewing design quality, separation of concerns, transaction boundaries, or structural maintainability. Triggers on "review architecture", "architecture review", "module boundaries", "transaction boundary", and related review requests.
license: MIT
metadata:
  author: Agent Skills Team
  version: "1.0.0"
---

# Code Review Architecture

Architecture review lens for design quality, coupling, cohesion, module boundaries, and rollback-safe side effects. Contains 8 rules across 8 focused categories for targeted multi-lens review work.

## When to Apply

Reference these guidelines when:
- Running a targeted review for this concern
- Merging findings from a broader multi-lens review
- Stress-testing a risky diff before approval
- Writing or refining repo-owned review instructions for this lens

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Coupling & Cohesion | CRITICAL | `arch-coupling-cohesion` |
| 2 | Separation of Concerns | CRITICAL | `arch-separation-concerns` |
| 3 | Abstraction & Interfaces | HIGH | `arch-abstraction-interfaces` |
| 4 | Module Boundaries | HIGH | `arch-module-boundaries` |
| 5 | Design Patterns | MEDIUM | `arch-design-patterns` |
| 6 | Data Flow | MEDIUM | `arch-data-flow` |
| 7 | Extensibility & Maintainability | MEDIUM | `arch-extensibility-maintainability` |
| 8 | Transaction Boundary Invariants | CRITICAL | `arch-transaction-boundaries` |

## Quick Reference

- `arch-coupling-cohesion` - Tight coupling, low cohesion, circular dependencies, dependency inversion issues
- `arch-separation-concerns` - Business logic placement, layering, and single-responsibility boundaries
- `arch-abstraction-interfaces` - Abstraction levels, interface quality, and leaky boundaries
- `arch-module-boundaries` - API contracts, visibility, and boundary enforcement
- `arch-design-patterns` - Appropriate pattern use, anti-patterns, and consistency
- `arch-data-flow` - Clear data movement, state handling, and persistence separation
- `arch-extensibility-maintainability` - Configurability, duplication, and future change safety
- `arch-transaction-boundaries` - Transactional writes versus external side effects and lifecycle-hook safety

## Scope Discipline

Apply this lens when explicitly requested or when a review workflow dispatches it.

### In Scope
- Review architectural concerns: design patterns, module structure, coupling, and boundaries
- Analyze separation of concerns and responsibility assignment
- Check abstraction levels, interfaces, and extension points
- Verify transactional safety when durable writes interact with external systems

### Out of Scope
- ❌ Type errors or type coverage issues
- ❌ Security vulnerabilities as the primary lens
- ❌ Timeout, retry, or exception hygiene details
- ❌ Raw throughput or micro-optimization analysis
- ❌ Pure readability or naming concerns

## Cross-Lens Handoff Discipline

Use this as a targeted lens, not a generic review bundle.

- Prove one dominant structural issue first.
- If another concern becomes primary, recommend exactly one next review lens instead of broadening into a vague multi-lens pass.
- Keep the current pass focused on evidence this lens can actually prove.

Smallest likely follow-up lenses:

- `code-review-performance` when the proven problem is mainly cost, query shape, or resource growth
- `code-review-error-handling` when rollback, cleanup, or partial-failure semantics are local rather than structural
- `code-review-security` when the boundary design mainly weakens isolation, authz, or trust controls
- `code-review-simplicity` when the root issue is over-abstraction more than architecture correctness

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

- **CRITICAL**: Architectural violations that can create rollback inconsistency, cascading failures, or long-lived technical debt.
- **HIGH**: Design flaws that materially hinder maintenance, extension, or safe change.
- **MEDIUM**: Structural improvements that would simplify evolution or reduce coupling.
- **LOW**: Minor architectural suggestions with limited risk.

## Metadata Guidance Tags

- **cross_lens_candidate** — true when the finding also suggests another lens should inspect the same location, otherwise false.
- **tradeoff_required** — true when the fix requires meaningful refactoring or behavior trade-offs, otherwise false.

## Adversarial Input Discipline

- Identify one plausible failure scenario for the main architectural change in the diff: rollback during a transaction, lifecycle hook under constraint failure, dependency cycle under load, or a contract change existing callers do not honor.
- For transactional or bulk-persistence diffs, name the transactional entry point, list every persisted type touched inside it, inspect non-post-commit lifecycle hooks on those types, and state whether any hook writes to a non-transactional store.
- If cross-file hook inspection is not performed when required, return BLOCKED instead of APPROVED.

## Integration Notes

- Part of the six-pass review protocol. Precedence: SECURITY > ERROR_HANDLING > TYPE_SAFETY > PERFORMANCE > ARCHITECTURE > SIMPLICITY.
- Dedupe merged findings by `(path, line, title)`.
- Lead with the highest-severity must-fix items first.

## References

- [Pi Ensemble architecture lens](https://raw.githubusercontent.com/randomm/pi-ensemble/main/skill/code-review-architecture/SKILL.md)
- [Martin Fowler on refactoring](https://refactoring.com/)
