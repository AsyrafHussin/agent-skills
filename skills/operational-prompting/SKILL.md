---
name: operational-prompting
description: Repo-owned operational prompting for coding agents. Use when designing AGENTS.md, Copilot/Codex instructions, path-specific instruction files, portable skill manifests, or validation contracts. Triggers on "operational prompting", "agent instructions", "repo-owned prompts", "Copilot instructions", or "portable agent skills".
license: MIT
metadata:
  author: Agent Skills Team
  version: "1.0.0"
---

# Operational Prompting

Modern coding-agent control for repository-bound work. Contains 8 rules across 4 categories for repo-owned instructions, instruction hierarchy, scope contracts, validation contracts, evidence output, machine-readable workflows, stopping conditions, and vendor-neutral skill manifests.

## When to Apply

Reference these guidelines when:
- Designing repository instructions for coding agents
- Migrating a giant custom prompt into repo-owned files
- Defining AGENTS.md, `.github/copilot-instructions.md`, or `*.instructions.md`
- Writing portable skill manifests or task-specific command recipes
- Tightening task boundaries, validation loops, or review output

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Repo-Owned Control | CRITICAL | `repo-`, `hierarchy-` |
| 2 | Task Contracts | CRITICAL | `scope-`, `stop-` |
| 3 | Validation & Evidence | HIGH | `validation-`, `evidence-`, `machine-` |
| 4 | Portability | MEDIUM | `portable-` |

## Quick Reference

### 1. Repo-Owned Control (CRITICAL)

- `repo-owned-instructions` - Put durable agent rules in repository files, not persona-heavy chat prompts
- `instruction-hierarchy` - Split rules across global, repo, path, and task layers with conflict resolution

### 2. Task Contracts (CRITICAL)

- `scope-minimal-diff` - Define the smallest affected file set and prohibit unrelated edits
- `stop-ask-conditions` - Stop when ambiguity, missing facts, or unsafe assumptions appear

### 3. Validation & Evidence (HIGH)

- `validation-contracts` - Bind work to exact repo commands, order, and stop-on-failure behavior
- `evidence-output` - Report files changed, commands run, failures, and unresolved risks
- `machine-readable-first` - Prefer JSON, lint, or other deterministic output before prose summaries

### 4. Portability (MEDIUM)

- `portable-skill-manifests` - Publish vendor-neutral skill manifests and task recipes in repo-owned YAML

## Essential Patterns

### Minimal Repository Contract

```markdown
# AGENTS.md

## Start here
- `README.md` for product behavior
- `.github/copilot-instructions.md` for repo-wide rules
- `src/` for implementation
- `tests/` for behavioral expectations

## Before editing
- Read the smallest affected file set first
- Run: composer test
- Run: composer phpstan
- Stop and ask if requirements conflict or the validation command is missing
```

### Vendor-Neutral Skill Manifest

```yaml
schemaVersion: 1
description: Repo-owned agent skills
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
  executionNotes:
    - Execute commands from the repository root unless a skill says otherwise.
    - Resolve paths relative to the repository root.
    - Prefer deterministic, machine-readable output when available.
skills:
  - id: validate-repo
    file: .ai/skills/validate-repo.yaml
    summary: Run the repository validation suite in a stable order.
```

## How to Use

Read individual rule files for detailed explanations, anti-patterns, and copy-pasteable examples.

Each rule file contains:
- YAML frontmatter with metadata (title, impact, tags)
- Why it matters
- Bad / Better / Best examples
- Exceptions and trade-offs
- Related topics for adjacent rules

## Full Compiled Document

For the complete guide with all rules expanded: `AGENTS.md`
