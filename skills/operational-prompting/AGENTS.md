# Operational Prompting - Complete Guide

**Version:** 1.0.0
**Focus:** Repo-owned instructions, scope contracts, validation contracts, evidence-first outputs
**Rules:** 8 (2 repo-owned + 2 task contracts + 3 validation/evidence + 1 portability)
**License:** MIT

---

## Overview

This skill helps agents replace persona-heavy prompt blobs with repository-bound execution contracts. Use it when creating or auditing `AGENTS.md`, `.github/copilot-instructions.md`, `*.instructions.md`, prompt files, or portable agent skill manifests.

## When to Use This Skill

Activate this skill when:
- Designing repository instructions for coding agents
- Migrating custom GPT/Copilot/Codex prompt gists into repo files
- Defining validation commands, stop conditions, or task boundaries
- Publishing portable, vendor-neutral skill manifests
- Auditing prompt systems for enforceability instead of style

## Trigger Phrases

The skill activates on:
- "operational prompting"
- "agent instructions"
- "repo-owned prompts"
- "Copilot instructions"
- "AGENTS.md"
- "portable agent skills"
- "validation contract"
- "task boundary"

## Maintenance Discipline

- Edit the canonical repo-owned instruction source first, not installed, copied, or generated derivatives.
- Regenerate machine-readable manifests or compiled prompt artifacts only when the interface-facing metadata actually changed.
- After changing repo-owned guidance, rerun the repository's validation and install/sync steps for the affected agent assets.
- When a durable correction appears only in chat history, memory, or review notes, promote it into the owning repo guidance instead of leaving it as tribal knowledge.
- Clean stale generated copies when a skill, prompt file, or subagent definition is renamed, merged, or retired.
- For one-off operational mutations, prefer inspect-first workflows with an explicit execute flag rather than auto-running destructive behavior by default.

## Rule Categories

### 1. Repo-Owned Control (CRITICAL - 2 rules)
**Prefix:** `repo-`, `hierarchy-`

- `repo-owned-instructions`
- `instruction-hierarchy`

### 2. Task Contracts (CRITICAL - 2 rules)
**Prefix:** `scope-`, `stop-`

- `scope-minimal-diff`
- `stop-ask-conditions`

### 3. Validation & Evidence (HIGH - 3 rules)
**Prefix:** `validation-`, `evidence-`, `machine-`

- `validation-contracts`
- `evidence-output`
- `machine-readable-first`

### 4. Portability (MEDIUM - 1 rule)
**Prefix:** `portable-`

- `portable-skill-manifests`

## Rule Index

| Rule | Category | Impact |
|------|----------|--------|
| `repo-owned-instructions` | Repo-Owned Control | CRITICAL |
| `instruction-hierarchy` | Repo-Owned Control | CRITICAL |
| `scope-minimal-diff` | Task Contracts | CRITICAL |
| `stop-ask-conditions` | Task Contracts | CRITICAL |
| `validation-contracts` | Validation & Evidence | HIGH |
| `evidence-output` | Validation & Evidence | HIGH |
| `machine-readable-first` | Validation & Evidence | HIGH |
| `portable-skill-manifests` | Portability | MEDIUM |

## Canonical-Source-First Workflow

1. Edit the reviewed source file, not generated copies.
2. Regenerate manifests or compiled assets only if needed.
3. Run the repo validation commands for the changed guidance surface.
4. Reinstall or resync generated agent assets.
5. Remind the user about any required client reload step only when that client actually needs one.

## Manual Operation Contract

1. Print the target objects or scope before any write step.
2. Default to dry-run or inspect mode.
3. Require an explicit `--execute` or equivalent flag for mutations.
4. Keep validation and rollback expectations visible in the task contract.

## Cross-Layer Task Contract

1. Name only the layers that actually change: data, domain, request boundary, rendering, browser flow, tests.
2. Assign one owner per changed layer so instructions do not overlap.
3. Sequence contract-producing layers before dependent layers.
4. Make the handoff explicit: accepted input, produced data shape, emitted IDs or values, proving tests.

## Specialist Routing Contract

1. Start with one primary specialist instruction set.
2. Add a second specialist only when the task clearly spans a second concern.
3. Avoid broad instruction bundles when a smaller owner can prove or implement the change.
4. After review or triage, hand off to the smallest proven next owner instead of escalating into another broad pass.

---

title: Use Repo-Owned Instructions Instead of Persona Blobs
impact: CRITICAL
impactDescription: Durable agent behavior must live in repository files
tags: repo, prompts, instructions, agents, governance
---

## Use Repo-Owned Instructions Instead of Persona Blobs

**Impact: CRITICAL**

Put durable agent rules in repository files that can be reviewed, versioned, and validated. A giant persona prompt can express preferences, but it cannot reliably bind an agent to the current repository, task boundary, file paths, or validation commands.

### Bad

```markdown
You are a genius principal engineer named X.
Always think harder.
Follow a 9-step ritual.
Do five hidden iterations.
```

### Better

```markdown
# AGENTS.md

## Start here
- `README.md`
- `src/`
- `tests/`

## Before editing
- Read the smallest affected file set
- Run `composer test`
- Stop and ask if requirements conflict
```

### Best

```markdown
# AGENTS.md

## Repository contract
- Use `README.md` for product behavior
- Use `.github/copilot-instructions.md` for repo-wide coding rules
- Use `apps/payments/*.instructions.md` for payment-specific constraints

## Before patching
- Read `composer.json`, `phpstan.neon`, `.php-cs-fixer.php`
- Limit changes to the smallest affected production/test file set
- Run `composer test -- --filter PaymentGateway`
- Run `composer phpstan`
- Stop if a required command is missing or the repo state contradicts the task
```

### Why

- Reviewable in normal code review
- Versioned with the codebase
- Easier to split by repo/path/task
- Prevents prompt drift disguised as personality

---

title: Layer Instructions by Global, Repo, Path, and Task Scope
impact: CRITICAL
impactDescription: Smaller instruction layers are easier to enforce and maintain
tags: hierarchy, instructions, scope, prompts, governance
---

## Layer Instructions by Global, Repo, Path, and Task Scope

**Impact: CRITICAL**

Do not collapse every rule into one file. Split stable rules by scope: tool/global defaults, repo-wide rules, path-specific constraints, and task-local prompts. Resolve conflicts explicitly instead of hoping a mega-prompt will win.

### Best Pattern

```text
Global agent defaults
  -> Repo contract (`AGENTS.md`, `.github/copilot-instructions.md`)
    -> Path rules (`src/payments/*.instructions.md`)
      -> Task prompt (today's request)
```

### Required Behavior

- Repo rules define shared workflow and validation
- Path rules override only local conventions
- Task prompts describe the current objective, not permanent policy
- Conflict order must be stated plainly

---

title: Define the Smallest Allowed Change Surface
impact: CRITICAL
impactDescription: Scope control prevents wandering refactors and collateral damage
tags: scope, minimal-diff, task-boundaries, edits
---

## Define the Smallest Allowed Change Surface

**Impact: CRITICAL**

Tell the agent what it may edit and what it must leave alone. "Be careful" is not a scope contract. A good scope contract names the smallest affected file set, the expected tests, and the classes of changes that are out of bounds.

### Bad

```markdown
Improve the codebase carefully and clean things up as needed.
```

### Better

```markdown
- Touch only the failing feature and its directly coupled tests.
- Do not reformat unrelated files.
- Do not rename APIs unless the task explicitly requires it.
```

### Best

```markdown
- First identify the smallest affected file set.
- Change only those production files and the tests needed to prove the fix.
- Do not refactor adjacent modules, update dependencies, or rename symbols outside the task boundary.
- If the fix requires a wider blast radius, stop and explain why before editing.
```

---

title: Stop and Ask on Ambiguity Instead of Guessing
impact: CRITICAL
impactDescription: Explicit stop conditions prevent confident, wrong edits
tags: stop-conditions, ambiguity, escalation, safety
---

## Stop and Ask on Ambiguity Instead of Guessing

**Impact: CRITICAL**

Every instruction system needs an escape hatch. If the agent cannot identify the target files, the validation command, or the desired behavior, it must stop and ask instead of hallucinating a solution.

### Stop Conditions

- Requirements conflict with repository state
- Multiple plausible implementations exist and the task does not choose one
- Validation commands are missing or fail for unrelated reasons
- Required secrets, external systems, or unavailable files block safe work
- The requested change would exceed the stated scope contract

### Required Response

Report the blocking fact, the missing decision, and the narrowest question needed to continue.

---

title: Bind Work to Exact Validation Commands
impact: HIGH
impactDescription: Validation contracts turn vague quality claims into executable gates
tags: validation, commands, ci, stop-on-failure
---

## Bind Work to Exact Validation Commands

**Impact: HIGH**

Do not tell an agent to "test everything" or "run static analysis" in the abstract. Tell it which command to run, in what order, and what to do when a command is missing or fails.

### Bad

```markdown
Make sure quality checks pass before finishing.
```

### Better

```markdown
- Run `composer test`
- Run `composer phpstan`
```

### Best

```markdown
## Validation contract
- Run `composer test -- --filter PaymentGatewayTest`
- Run `composer phpstan`
- Run `composer cs:check`
- Stop on the first failing command
- Report the failing command verbatim and do not claim success
```

### Why

- Reviewers can reproduce the same checks
- Agents stop hiding behind generic "validated" language
- CI and local workflows stay aligned

---

title: Require Evidence-First Summaries
impact: HIGH
impactDescription: Evidence keeps review output auditable and falsifiable
tags: evidence, summaries, review, audit
---

## Require Evidence-First Summaries

**Impact: HIGH**

A good agent summary shows what changed and how it was verified. Ritual narration, hidden loops, and theatrical "system 2" framing do not help reviewers. Evidence does.

### Include in Final Output

- Files changed
- Commands run
- Result of each command
- Tests added or updated
- Remaining risks or unresolved follow-ups

### Example

```markdown
Changed:
- `src/Payments/Gateway.php`
- `tests/Payments/GatewayTest.php`

Validation:
- `composer test -- --filter GatewayTest` ✅
- `composer phpstan` ✅

Risk:
- No staging credential was available to verify the live provider handshake.
```

---

title: Prefer Machine-Readable Output Before Prose
impact: HIGH
impactDescription: Deterministic output reduces interpretation drift
tags: json, deterministic-output, lint, agents, tooling
---

## Prefer Machine-Readable Output Before Prose

**Impact: HIGH**

When a tool can emit JSON, lint-style, or other deterministic output, prefer that first and summarize second. Machine-readable output is easier to rerun, diff, aggregate, and feed into follow-up tools.

### Good Pattern

```markdown
- Use `--json` for agent consumption
- Use `--lint` or line-oriented output for quick human review
- Keep the original evidence text and locations intact in the summary
```

### Why

- Less summarization drift
- Better automation in CI and PR review bots
- Easier grouping by severity, path, rule, or repeated identifier

---

title: Publish Portable Skill Manifests in the Repository
impact: MEDIUM
impactDescription: Vendor-neutral manifests let multiple agents share the same task recipes
tags: portability, yaml, skills, manifests, copilot, codex
---

## Publish Portable Skill Manifests in the Repository

**Impact: MEDIUM**

If a repository relies on repeatable agent tasks, capture them in repo-owned YAML instead of burying them in one vendor's UI. A portable manifest makes the same task recipe available to Copilot, Codex, Claude, Gemini, or any tool that can read files.

### Best

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
  executionNotes:
    - Execute commands from the repository root unless a skill says otherwise.
    - Resolve paths relative to the repository root.
    - Prefer deterministic, machine-readable output when a skill offers multiple output modes.
skills:
  - id: validate-repo
    file: .ai/skills/validate-repo.yaml
    summary: Run the repository validation suite in a stable order.
```

```yaml
name: validate-repo
purpose: Run the standard validation suite for this repository.
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

### Why

- Shared source of truth across tools
- Easier code review than UI-only prompts
- Safer than copying task recipes between vendors by hand
