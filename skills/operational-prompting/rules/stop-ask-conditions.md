---
title: Stop and Ask on Ambiguity Instead of Guessing
impact: CRITICAL
impactDescription: Explicit stop conditions prevent confident, wrong edits
tags: stop-conditions, ambiguity, escalation, safety
---

# Stop and Ask on Ambiguity Instead of Guessing

## Why it matters

An agent that never stops will invent missing facts, commands, or requirements. Explicit escalation rules are cheaper than cleaning up a confident, wrong patch.

## Rule

Define clear stop conditions for ambiguity, missing repo facts, unavailable validation commands, external blockers, and scope expansion. Require a focused question instead of speculative edits.

## Bad

```markdown
If something is unclear, use your best judgment and keep going.
```

## Better

```markdown
Stop and ask if the required file or command cannot be found.
```

## Best

```markdown
Stop immediately when:
- two plausible implementations exist
- validation commands are missing
- repo state contradicts the request
- the change exceeds the declared scope
Ask one narrow question that unblocks the work.
```

## Exceptions / trade-offs

Routine implementation details do not need user confirmation when the repository already defines the pattern clearly. Stop conditions are for missing or conflicting facts, not for every small choice.

## Related topics

- [scope-minimal-diff.md](scope-minimal-diff.md) — scope drift is a stop condition
- [evidence-output.md](evidence-output.md) — report blockers and unresolved risks plainly
