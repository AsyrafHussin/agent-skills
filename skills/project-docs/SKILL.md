---
name: project-docs
description: Project documentation lifecycle for PHP/Laravel and Node/TypeScript/React projects — bootstrapping essential docs, naming and folder conventions, freshness, and cleanup of AI-generated junk and stale files. Use when starting a new project, setting up docs/ structure, auditing markdown files, cleaning up the docs folder, or deciding which docs to keep, archive, or delete. Triggers on "set up docs", "audit docs", "clean up markdown", "what docs does this project need", "organize docs folder", "find stale docs".
license: MIT
metadata:
  author: agent-skills
  version: "1.0.0"
---

# Project Documentation

End-to-end documentation lifecycle for PHP/Laravel and Node/TypeScript/React projects. Contains 25 rules across 6 categories covering folder structure, naming conventions, essential files, content quality (including AI-slop detection), cleanup of accumulated junk, and lifecycle. Supports **bootstrap mode** (set up docs in a new project), **audit mode** (find what's missing, stale, bloated, or junk), and **reference mode** (conventions lookup).

## Metadata

- **Version:** 1.0.0
- **Scope:** PHP / Laravel + Node / TypeScript / React projects
- **Rule Count:** 25 rules across 6 categories
- **License:** MIT

## How to Use This Skill — Three Modes

### Mode 1: Bootstrap (new project / missing docs)

When the user asks "set up docs", "what docs does this project need", or starts a new project — walk through the bootstrap steps:

1. **Detect project type** (Laravel? React? Both?) — `composer.json`, `package.json`, `artisan` binary
2. **Inventory existing docs** — list every `.md` file with its location and last-modified date
3. **Identify gaps** — compare against the Essential Files checklist; report what's missing
4. **Propose folder structure** — `docs/` with sub-folders (`architecture/`, `adr/`, `guides/`, `runbooks/`, `archive/`) based on project size
5. **Offer to scaffold templates** — README, CHANGELOG, LICENSE, CONTRIBUTING, SECURITY, ADR-0001 — generate with user approval, do not auto-create
6. **Suggest CI gates** — markdown-lint, broken-link checker (lychee), CHANGELOG-on-PR enforcement

### Mode 2: Audit (existing project cleanup)

When the user asks "audit docs", "clean up markdown", or "what should I delete" — produce a classified ledger.

For each `.md` file in the repo:
- **KEEP** — file is essential and current
- **UPDATE** — file is essential but stale (e.g., README contradicts current setup)
- **ARCHIVE** — superseded but historically useful — move to `docs/archive/<year>/`
- **DELETE** — AI-generated plan files, empty stubs, duplicates, orphaned drafts
- **MOVE** — wrong location or wrong name (e.g., `MyArchitectureNotes.md` at root → `docs/architecture/overview.md`)

Output format:
```
## Documentation Audit Ledger

| File | Last modified | Verdict | Reason | Action |
|------|---------------|---------|--------|--------|
| PLAN.md | 2026-02-14 | DELETE | AI-generated plan, no longer referenced | rm PLAN.md |
| README.md | 2024-08-01 | UPDATE | Setup steps reference removed Vite v3 | Update install section |
| docs/old-architecture.md | 2024-11 | ARCHIVE | Superseded by docs/architecture/overview.md | mv to docs/archive/2024/ |
| MyNotes.md | 2025-09 | DELETE | Personal notes; not project docs | rm MyNotes.md |

## Summary
- KEEP: X files
- UPDATE: Y files (top priority: ...)
- ARCHIVE: Z files
- DELETE: N files
- MOVE: M files
```

**Never auto-delete.** Always surface for user approval first.

### Mode 3: Reference (conventions lookup)

When the user asks "how should I name this", "where should this go", or references the skill in a code-review context — look up the relevant rule(s) in `rules/`.

## When to Apply

Reference this skill when:
- Starting a new Laravel or Node/React project and need a docs baseline
- Onboarding a project with messy or AI-cluttered markdown files
- Setting up `docs/` folder structure
- Naming a new doc file
- Deciding whether to delete a `PLAN.md` / `TODO.md` / `IMPLEMENTATION-SUMMARY.md`
- Adding CI checks for markdown quality
- Reviewing a PR that adds or modifies documentation
- Quarterly "docs hygiene" sweep

## Step 1: Detect Project Type

**Always check the project stack before recommending specifics.** Bootstrap and naming guidance differ slightly per stack.

| Signal | Project Type | Notes |
|--------|--------------|-------|
| `composer.json` + `artisan` | Laravel (PHP) | README should cover `composer install`, `php artisan migrate`, `.env.example` |
| `package.json` (only) | Node / TypeScript / React | README should cover `npm install`, `.nvmrc`, build scripts |
| Both present | Laravel + Inertia + React | README covers both PHP and Node setup paths |

The rules themselves are mostly stack-agnostic — README format, ADR structure, naming conventions apply to any project.

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Structure | CRITICAL | `structure-` |
| 2 | Naming | CRITICAL | `naming-` |
| 3 | Essential Files | HIGH | `essential-` |
| 4 | Quality | HIGH | `quality-` |
| 5 | Cleanup | HIGH | `cleanup-` |
| 6 | Lifecycle | MEDIUM | `lifecycle-` |

## Quick Reference

### 1. Structure (CRITICAL)

- `structure-root-files` — Which files belong at the repo root (README, CHANGELOG, LICENSE, etc.)
- `structure-docs-folder` — `docs/` as the home for everything beyond root files
- `structure-subfolders` — Recommended `docs/` layout: architecture/, adr/, guides/, runbooks/, archive/

### 2. Naming (CRITICAL)

- `naming-root-files` — `UPPERCASE.md` for conventional root files
- `naming-docs-files` — `kebab-case.md` for files under `docs/`
- `naming-adr-files` — Numbered prefix: `0001-record-architecture-decisions.md`
- `naming-anti-patterns` — No dates, no `MyNotes.md`, no `tmp/draft/final` markers

### 3. Essential Files (HIGH)

- `essential-readme` — Every project needs a README with purpose, install, usage, license
- `essential-changelog` — Keep-a-Changelog format; one entry per release
- `essential-license` — `LICENSE` file (or `LICENSE.md`) at repo root
- `essential-contributing` — `CONTRIBUTING.md` when accepting external contributors
- `essential-security` — `SECURITY.md` with vulnerability reporting policy

### 4. Quality (HIGH)

- `quality-conciseness` — Cut bloat; length is a cost, not a virtue
- `quality-ai-slop` — Detect AI-generated content patterns (filler, sign-offs, generic praise)
- `quality-headings` — One H1, no skipped levels, descriptive heading text
- `quality-code-blocks` — Language tags, copy-pasteable commands, no untagged blocks
- `quality-links` — Descriptive link text (not "click here"), relative paths, no broken links

### 5. Cleanup (HIGH)

- `cleanup-ai-junk` — Detect and remove AI-generated plan/summary files
- `cleanup-duplicates` — Same content in multiple files; consolidate or delete copies
- `cleanup-orphans` — `.md` files not linked from anywhere; archive or delete
- `cleanup-empty-stubs` — Files with TBD / TODO / placeholder content only

### 6. Lifecycle (MEDIUM)

- `lifecycle-freshness` — "Last verified" dates on architecture docs
- `lifecycle-archive` — Superseded docs go to `docs/archive/<year>/`
- `lifecycle-adr-process` — ADR creation triggers and lifecycle (proposed → accepted → superseded)
- `lifecycle-changelog-discipline` — Add a CHANGELOG entry in the same PR as the change

## Essential Patterns

### Standard folder layout

```
.
├── README.md                      # required
├── CHANGELOG.md                   # required from first release
├── LICENSE                        # required
├── CONTRIBUTING.md                # if external contributors
├── SECURITY.md                    # if internet-facing
├── CODE_OF_CONDUCT.md             # if open source community
├── .github/
│   └── CODEOWNERS                 # team ownership
└── docs/
    ├── architecture/
    │   ├── overview.md
    │   └── data-model.md
    ├── adr/
    │   ├── 0001-record-architecture-decisions.md
    │   ├── 0002-choose-mysql-over-postgres.md
    │   └── 0003-adopt-inertia-for-spa.md
    ├── guides/
    │   ├── getting-started.md
    │   ├── deployment.md
    │   └── local-development.md
    ├── runbooks/
    │   ├── deploy-production.md
    │   └── incident-response.md
    └── archive/
        └── 2024/
            └── old-architecture-notes.md
```

### File naming at a glance

```
✓ README.md, CHANGELOG.md, LICENSE, CONTRIBUTING.md, SECURITY.md
✓ docs/architecture/overview.md
✓ docs/adr/0007-cache-strategy.md
✓ docs/guides/deployment.md
✓ docs/archive/2024/q3-launch-plan.md

✗ Readme.md, Changelog.md          (use UPPERCASE for conventional root files)
✗ docs/Architecture/Overview.md    (use kebab-case in docs/)
✗ docs/Notes-2025-09-14.md         (no dates in filenames)
✗ MyArchitectureThoughts.md        (no first-person, no PascalCase)
✗ PLAN.md, TODO.md, TEMP.md        (use issue tracker for transient state)
✗ FINAL-deployment-guide-v2.md     (no draft/final/v2 markers)
```

### CI: keep markdown honest

```yaml
# .github/workflows/docs.yml
- name: Lint markdown
  uses: DavidAnson/markdownlint-cli2-action@v23

- name: Check links
  uses: lycheeverse/lychee-action@v2
  with:
    args: --no-progress --exclude-mail './**/*.md'
```

## How to Use

Read individual rule files for detailed conventions and examples:

```
rules/structure-root-files.md
rules/naming-adr-files.md
rules/essential-readme.md
rules/cleanup-ai-junk.md
rules/lifecycle-archive.md
```

Each rule file contains:
- YAML frontmatter with metadata (title, impact, tags)
- Brief explanation of why it matters
- Incorrect example (anti-pattern)
- Correct example (the convention)
- Detection / enforcement guidance where applicable

## References

- [Keep a Changelog](https://keepachangelog.com/)
- [Semantic Versioning](https://semver.org/)
- [Architecture Decision Records (ADR)](https://adr.github.io/)
- [Diátaxis — documentation framework](https://diataxis.fr/)
- [Choose a License](https://choosealicense.com/)
- [Markdownlint](https://github.com/DavidAnson/markdownlint)
- [Lychee — link checker](https://github.com/lycheeverse/lychee)

## Full Compiled Document

For the complete guide with all rules expanded: `AGENTS.md`
