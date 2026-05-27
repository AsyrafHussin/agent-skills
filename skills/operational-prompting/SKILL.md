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

## Maintenance Discipline

- Edit the canonical repo-owned instruction source first, not installed, copied, or generated derivatives.
- Regenerate machine-readable manifests or compiled prompt artifacts only when the interface-facing metadata actually changed.
- After changing repo-owned guidance, rerun the repository's validation and install/sync steps for the affected agent assets.
- When a durable correction appears only in chat history, memory, or review notes, promote it into the owning repo guidance instead of leaving it as tribal knowledge.
- Clean stale generated copies when a skill, prompt file, or subagent definition is renamed, merged, or retired.
- For one-off operational mutations, prefer inspect-first workflows with an explicit execute flag rather than auto-running destructive behavior by default.

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

### Canonical-Source-First Workflow

```markdown
1. Edit the reviewed source file, not generated copies
2. Regenerate manifests or compiled assets only if needed
3. Run the repo validation commands for the changed guidance surface
4. Reinstall or resync generated agent assets
5. Remind the user about any required client reload step only when that client actually needs one
```

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

### Manual Operation Contract

```markdown
## One-off operation
- Print the target objects or scope before any write step
- Default to dry-run / inspect mode
- Require an explicit `--execute` or equivalent flag for mutations
- Keep validation and rollback expectations visible in the task contract
```

### Cross-Layer Task Contract

```markdown
## Cross-layer change
- Name only the layers that actually change (data, domain, request boundary, rendering, browser flow, tests)
- Assign one owner per changed layer so instructions do not overlap
- Sequence contract-producing layers before dependent layers
- Make the handoff explicit: accepted input, produced data shape, emitted IDs/values, proving tests
```

### Specialist Routing Contract

```markdown
## Specialist routing
- Start with one primary specialist instruction set
- Add a second specialist only when the task clearly spans a second concern
- Avoid broad instruction bundles when a smaller owner can prove or implement the change
- After review or triage, hand off to the smallest proven next owner instead of escalating into another broad pass
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
