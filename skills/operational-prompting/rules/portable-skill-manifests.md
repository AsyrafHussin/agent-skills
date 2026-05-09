---
title: Publish Portable Skill Manifests in the Repository
impact: MEDIUM
impactDescription: Vendor-neutral manifests let multiple agents share the same task recipes
tags: portability, yaml, skills, manifests, copilot, codex
---

# Publish Portable Skill Manifests in the Repository

## Why it matters

If a task recipe lives only inside one vendor UI, every other tool has to rediscover or re-copy it. Repo-owned skill manifests make workflows reviewable, portable, and versioned with the codebase.

## Rule

Store reusable agent task recipes in repo-owned YAML with explicit triggers, environment requirements, commands, outputs, and failure handling. Prefer one manifest that indexes the available skills.

## Bad

```markdown
The only copy of the workflow lives in a chat preset or product-specific settings screen.
```

## Better

```yaml
name: validate-repo
purpose: Run the repository validation suite.
commands:
  - run: composer test
```

## Best

```yaml
schemaVersion: 1
description: Vendor-neutral skill manifest for coding agents working in this repository.
skillSchema:
  requiredFields:
    - name
    - purpose
    - triggers
    - requiredEnvironment
    - inputs
    - commands
    - expectedOutputs
    - failureHandling
skills:
  - id: validate-repo
    file: .ai/skills/validate-repo.yaml
    summary: Run the repository validation suite in a stable order.
```

```yaml
name: validate-repo
purpose: Run the repository validation suite in a stable order.
triggers:
  - validate this repo
requiredEnvironment:
  workingDirectory: repository root
inputs:
  required: []
commands:
  - description: Run tests
    run: composer test
expectedOutputs:
  - Successful completion of each command in order.
failureHandling:
  - Stop on the first failing command and report it verbatim.
```

## Exceptions / trade-offs

Very small repositories may not need a full manifest yet. Even then, keep reusable task recipes in plain files inside the repo, not in private UI settings.

## Related topics

- [repo-owned-instructions.md](repo-owned-instructions.md) — keep durable policy in files
- [machine-readable-first.md](machine-readable-first.md) — portable skills should declare preferred output modes
