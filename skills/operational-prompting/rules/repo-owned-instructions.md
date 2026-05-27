---
title: Use Repo-Owned Instructions Instead of Persona Blobs
impact: CRITICAL
impactDescription: Durable agent behavior must live in repository files
tags: repo, prompts, instructions, agents, governance
---

# Use Repo-Owned Instructions Instead of Persona Blobs

## Why it matters

Persona-heavy mega-prompts are hard to review, impossible to diff cleanly, and detached from the repository facts that actually constrain good work. Durable agent behavior belongs in repository files that travel with the codebase.

## Rule

Put stable agent rules in `AGENTS.md`, `.github/copilot-instructions.md`, `*.instructions.md`, or repo-owned prompt files. Use the task prompt only for the current objective, not as the permanent control plane.

## Bad

```markdown
You are a genius architect.
Always think harder than everyone else.
Do hidden iterations until the code feels perfect.
```

## Better

```markdown
# AGENTS.md

## Start here
- `README.md`
- `src/`
- `tests/`
```

## Best

```markdown
# AGENTS.md

## Repository contract
- `README.md` defines product behavior
- `.github/copilot-instructions.md` defines repo-wide coding rules
- `src/payments/*.instructions.md` defines payment-specific constraints
```

## Exceptions / trade-offs

Ephemeral task context still belongs in the active task prompt. The rule is about keeping durable policy out of chat-only personas, not about banning task descriptions.

## Related topics

- [instruction-hierarchy.md](instruction-hierarchy.md) — split repo, path, and task rules cleanly
- [portable-skill-manifests.md](portable-skill-manifests.md) — publish reusable task recipes in YAML
