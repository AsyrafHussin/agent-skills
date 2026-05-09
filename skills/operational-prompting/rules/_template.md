---
title: Rule Title Here
impact: MEDIUM
impactDescription: Optional description of impact
tags: tag1, tag2, tag3
---

# Rule Title Here

## Impact level guide

- **CRITICAL** — Safety, repo-owned control, task boundaries, or stop/ask rules that must hold before any other guidance.
- **HIGH** — Validation, evidence, or workflow rules that materially affect correctness, reviewability, or reproducibility.
- **MEDIUM** — Portability, maintainability, or structure rules that improve consistency without changing the core safety contract.
- **LOW** — Optional polish, wording, or convenience guidance with limited operational risk.

## Why it matters

One short paragraph on what becomes unreliable, unreviewable, or hard to verify when this rule is ignored.

## Rule

A blunt, testable recommendation in one or two sentences.

## Bad

```markdown
# Anti-pattern example
```

## Better

```markdown
# Improved example
```

## Best

```markdown
# Preferred repo-owned pattern
```

## Exceptions / trade-offs

List the narrow cases where the rule does not apply.

## Related topics

- [related-rule.md](related-rule.md) — one-line description of the relationship
