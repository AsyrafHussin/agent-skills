---
title: Define the Smallest Allowed Change Surface
impact: CRITICAL
impactDescription: Scope control prevents wandering refactors and collateral damage
tags: scope, minimal-diff, task-boundaries, edits
---

# Define the Smallest Allowed Change Surface

## Why it matters

Agents drift when a task says "clean things up" without naming the boundary. A scope contract keeps the fix reviewable and avoids surprise refactors, dependency churn, or style edits outside the task.

## Rule

Tell the agent to identify the smallest affected file set, edit only what is directly coupled to the task, and ask before widening the blast radius.

## Bad

```markdown
Improve this area and clean up anything nearby that looks wrong.
```

## Better

```markdown
- Touch only the failing feature and its directly coupled tests.
- Do not reformat unrelated files.
```

## Best

```markdown
- First identify the smallest affected production and test file set.
- Limit edits to those files unless the task explicitly expands scope.
- Stop and explain why if the fix requires touching adjacent modules.
```

## Exceptions / trade-offs

Some repo-wide migrations legitimately span many files. In that case, the task must say so explicitly and define the allowed file classes up front.

## Related topics

- [stop-ask-conditions.md](stop-ask-conditions.md) — stop when the required scope becomes unclear
- [validation-contracts.md](validation-contracts.md) — scope should map to explicit validation
