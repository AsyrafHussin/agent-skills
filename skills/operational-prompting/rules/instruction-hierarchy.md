---
title: Layer Instructions by Global, Repo, Path, and Task Scope
impact: CRITICAL
impactDescription: Smaller instruction layers are easier to enforce and maintain
tags: hierarchy, instructions, scope, prompts, governance
---

# Layer Instructions by Global, Repo, Path, and Task Scope

## Why it matters

One giant prompt mixes stable policy with local exceptions and one-off task goals. Instruction layering keeps the stable rules stable and makes overrides visible instead of accidental.

## Rule

Split rules by scope: global agent defaults, repository-wide instructions, path-specific constraints, and task-local prompts. State the conflict order so agents and reviewers know which layer wins.

## Bad

```markdown
One 2,000-line prompt containing global defaults, repo rules, payment rules, and today's bugfix instructions.
```

## Better

```text
Global defaults
  -> Repo rules
    -> Path rules
      -> Task prompt
```

## Best

```markdown
## Rule priority
1. Safety and security
2. System and developer constraints
3. User request
4. Repo workflow rules
5. Style preferences
```

## Exceptions / trade-offs

Tiny repositories may not need path-specific files yet. Even then, keep repo rules separate from task prompts so the structure can grow cleanly.

## Related topics

- [repo-owned-instructions.md](repo-owned-instructions.md) — stable policy belongs in files
- [scope-minimal-diff.md](scope-minimal-diff.md) — task-local prompts still need explicit boundaries
