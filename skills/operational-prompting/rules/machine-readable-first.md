---
title: Prefer Machine-Readable Output Before Prose
impact: HIGH
impactDescription: Deterministic output reduces interpretation drift
tags: json, deterministic-output, lint, agents, tooling
---

# Prefer Machine-Readable Output Before Prose

## Why it matters

Machine-readable output can be re-run, diffed, grouped, and consumed by follow-up tools. Prose-only summaries lose structure and invite subtle distortion.

## Rule

When a command supports JSON, lint-style, or other deterministic output, prefer that output for the primary evidence and summarize it second.

## Bad

```markdown
Run the tool normally and paraphrase whatever it says.
```

## Better

```markdown
Use `--json` for agent review and `--lint` for quick terminal inspection.
```

## Best

```markdown
- Generate JSON findings first
- Preserve rule IDs, severity, paths, and locations
- Summarize patterns only after the raw evidence is available
```

## Exceptions / trade-offs

Some small tools have only human-readable output. In that case, keep the raw tool output close to the summary instead of over-paraphrasing it.

## Related topics

- [evidence-output.md](evidence-output.md) — summaries should remain auditable
- [portable-skill-manifests.md](portable-skill-manifests.md) — portable skills should declare preferred output modes
