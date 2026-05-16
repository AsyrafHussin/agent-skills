# Project Documentation - Complete Reference

**Version:** 1.0.0
**Organization:** Agent Skills Contributors
**Date:** May 2026
**License:** MIT

## Abstract

End-to-end documentation lifecycle for **PHP/Laravel and Node/TypeScript/React** projects. Contains 25 rules across 6 categories covering folder structure, naming conventions, essential files, content quality (including AI-slop detection in prose), cleanup of accumulated junk files, and the lifecycle from creation to archive. Supports three modes: **bootstrap** (set up docs in a new project), **audit** (find what's missing, stale, bloated, or junk), and **reference** (look up a convention). Each rule includes detection commands and worked examples for both stacks.

## How to Use

When asked to "set up docs", "audit docs", or "clean up markdown", produce a classified ledger of every `.md` file in the repo with a verdict (KEEP / UPDATE / ARCHIVE / DELETE / MOVE), a reason, and the recommended action. Never auto-delete — always surface for user approval. For bootstrap, propose the folder structure and offer to scaffold templates for missing essential files. For content review, surface AI-slop fingerprints and bloat candidates for trimming.

## References

- [Keep a Changelog](https://keepachangelog.com/)
- [Semantic Versioning](https://semver.org/)
- [Architecture Decision Records (ADR)](https://adr.github.io/)
- [Diátaxis — documentation framework](https://diataxis.fr/)
- [Choose a License](https://choosealicense.com/)
- [Markdownlint](https://github.com/DavidAnson/markdownlint)
- [Lychee — link checker](https://github.com/lycheeverse/lychee)

## Step 1: Detect Project Type

Inspect `composer.json`, `package.json`, and `artisan` binary to identify the stack.

| Signal | Project Type | Notes |
|--------|--------------|-------|
| `composer.json` + `artisan` | Laravel (PHP) | README covers `composer install`, `php artisan migrate` |
| `package.json` only | Node / TypeScript / React | README covers `npm install`, `.nvmrc`, build scripts |
| Both present | Laravel + Inertia + React | README covers both setup paths |

Rules themselves are mostly stack-agnostic — README format, ADR structure, naming conventions, and content-quality rules apply to any project.

---

# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. Structure (structure)

**Impact:** CRITICAL
**Description:** Where documentation lives in the repo — which files belong at the root, and how the `docs/` folder is organized. Structure decisions made early shape every future docs choice; getting them wrong scatters knowledge across the tree.

## 2. Naming (naming)

**Impact:** CRITICAL
**Description:** Consistent file naming so readers can find docs without guessing — `UPPERCASE.md` for conventional root files, `kebab-case.md` for everything in `docs/`, numbered prefixes for ADRs, and an explicit list of anti-patterns (dates, draft markers, first-person names) to avoid.

## 3. Essential Files (essential)

**Impact:** HIGH
**Description:** The minimum set every project needs (README, CHANGELOG, LICENSE) plus situational additions (CONTRIBUTING for external contributors, SECURITY for internet-facing services). These files set first impressions, legal posture, and incident response paths.

## 4. Quality (quality)

**Impact:** HIGH
**Description:** Content quality inside docs — conciseness over bloat, detection of AI-generated slop patterns in prose (filler phrases, generic praise, closing sign-offs), valid heading hierarchy, copy-pasteable code blocks, and descriptive non-broken links. Where Cleanup removes whole junk files, Quality cuts junk inside otherwise-legitimate docs.

## 5. Cleanup (cleanup)

**Impact:** HIGH
**Description:** Identifying junk that accumulates — AI-generated plan/summary files, near-duplicates of the same content, orphaned drafts nobody links to, and empty stubs. Cleanup is triaged, never automatic, so the user keeps final say on every delete.

## 6. Lifecycle (lifecycle)

**Impact:** MEDIUM
**Description:** How docs are created, kept current, archived, and replaced. Covers freshness dates on architecture docs, the archive workflow, ADR proposed → accepted → superseded states, and the discipline of updating CHANGELOG in the same PR as the change.

---


## Root-Level Documentation Files

**Impact: CRITICAL (Discoverability — first impression for every new reader, every tool, every hosting platform)**

A small, curated set of files lives at the repo root because GitHub, npm, Packagist, and every developer tool look for them there. Anything else belongs in `docs/`. Keeping the root clean turns the first directory listing into a clear signpost rather than a clutter dump.

## Incorrect

```
❌ Cluttered root — every doc lives at the top level
.
├── README.md
├── CHANGELOG.md
├── LICENSE
├── architecture.md
├── deployment-notes.md
├── PLAN.md
├── refactor-thoughts.md
├── api-reference.md
├── adr-001.md
├── adr-002.md
├── deployment-runbook.md
├── onboarding.md
└── ...
```

**Problems:**
- A new contributor scrolling the root sees 20+ files and can't tell what's essential
- Architecture and runbooks bury the README and CHANGELOG
- Each new doc adds noise at the most visible location in the repo

## Correct

```
✅ Curated root — only conventional files; everything else in docs/
.
├── README.md                  # required — project purpose, install, usage
├── CHANGELOG.md               # required from first release
├── LICENSE                    # required
├── CONTRIBUTING.md            # if accepting external contributors
├── SECURITY.md                # if internet-facing service
├── CODE_OF_CONDUCT.md         # if open-source community
└── docs/                      # everything else
    ├── architecture/
    ├── adr/
    ├── guides/
    └── runbooks/
```

**Benefits:**
- The root listing reads as a project index, not a junk drawer
- Tools (GitHub, npm, Packagist, Dependabot) find the files they expect
- New contributors see the curated set first; details live one click deeper in `docs/`

## Allowed root-level docs

| File | When required | What it contains |
|---|---|---|
| `README.md` | Always | Purpose, install, usage, license link |
| `CHANGELOG.md` | After first release | One section per version (Keep-a-Changelog format) |
| `LICENSE` | Always | The license text — plain, no `.md` extension by convention |
| `CONTRIBUTING.md` | If external contributors | Branching, commit style, PR checklist |
| `SECURITY.md` | If internet-facing | Vulnerability reporting policy and contact |
| `CODE_OF_CONDUCT.md` | Open-source projects | Contributor Covenant or equivalent |
| `AUTHORS.md` / `MAINTAINERS.md` | Multi-author projects | Maintainer list |

Anything else — architecture, ADRs, guides, runbooks, API references — belongs in `docs/`.

## Detection

```bash
# Files at root that aren't on the allow-list
ls *.md *.MD LICENSE 2>/dev/null | \
  grep -vE '^(README|CHANGELOG|CONTRIBUTING|SECURITY|CODE_OF_CONDUCT|AUTHORS|MAINTAINERS|LICENSE)(\.md)?$'
```

Reference: [GitHub — About READMEs](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes) · [Keep a Changelog](https://keepachangelog.com/)

---


## docs/ as the Single Documentation Home

**Impact: CRITICAL (One predictable home for everything beyond root files — no more "where did we put that diagram?")**

Every project needs one canonical place for documentation that isn't a conventional root file. `docs/` is the standard, recognized by GitHub Pages, MkDocs, Docusaurus, and most static-site generators. Without it, docs scatter across `wiki/`, `docs-internal/`, `notes/`, and inline `.md` files next to source.

## Incorrect

```
❌ Docs scattered across the tree
.
├── README.md
├── architecture-notes.md         # at root
├── deployment.md                 # at root
├── app/Services/PaymentService.md     # next to source
├── notes/
│   ├── design-meeting-2024.md
│   └── refactor-ideas.md
├── wiki/                         # parallel docs folder
│   ├── onboarding.md
│   └── api.md
└── internal-docs/
    └── runbook.md
```

**Problems:**
- Same kind of content (deployment, runbooks) lives in 3 different folders
- Static-site generators (MkDocs, Docusaurus, GitHub Pages) expect `docs/` and won't find the others
- New contributors don't know where to put a new doc — and don't know where to find existing ones

## Correct

```
✅ Everything non-root lives under docs/
.
├── README.md
├── CHANGELOG.md
├── LICENSE
└── docs/
    ├── architecture/
    │   └── overview.md
    ├── adr/
    │   └── 0001-record-architecture-decisions.md
    ├── guides/
    │   ├── getting-started.md
    │   └── deployment.md
    ├── runbooks/
    │   └── incident-response.md
    └── archive/
        └── 2024/
            └── superseded-design.md
```

**Benefits:**
- One predictable location — readers know where to look, contributors know where to add
- Works out of the box with GitHub Pages (`/docs` source), MkDocs, Docusaurus
- Source code stays uncluttered; no `.md` files next to `.php`/`.tsx` files

## Exceptions

A few `.md` files genuinely belong outside `docs/`:

- **Root-level conventional files** — README, CHANGELOG, LICENSE, etc. (see `structure-root-files`)
- **GitHub-specific files** — `.github/CODEOWNERS`, `.github/PULL_REQUEST_TEMPLATE.md`, `.github/ISSUE_TEMPLATE/*.md`
- **Top-level subproject READMEs in monorepos** — `packages/auth/README.md` is fine; the package itself is a unit

Everything else: in `docs/`.

## Detection

```bash
# Find .md files that aren't in docs/ or one of the allowed locations
find . -name '*.md' -type f \
  -not -path './docs/*' \
  -not -path './.github/*' \
  -not -path './node_modules/*' \
  -not -path './vendor/*' \
  -not -name 'README.md' \
  -not -name 'CHANGELOG.md' \
  -not -name 'CONTRIBUTING.md' \
  -not -name 'SECURITY.md' \
  -not -name 'CODE_OF_CONDUCT.md' \
  -not -name 'AUTHORS.md' \
  -not -name 'MAINTAINERS.md'
```

Any hit is a candidate for relocation into `docs/` (or deletion if it's junk).

Reference: [Diátaxis — documentation framework](https://diataxis.fr/) · [MkDocs](https://www.mkdocs.org/)

---


## docs/ Sub-folder Layout

**Impact: HIGH (Different doc kinds have different audiences and lifecycles — separate them)**

A flat `docs/` works for 5 files and falls apart at 20. Sub-folders by *purpose* (architecture, ADRs, guides, runbooks, archive) make docs scannable and let you apply different freshness/ownership rules per folder.

## Incorrect

```
❌ Flat docs/ — everything mixed together
docs/
├── overview.md
├── deployment.md
├── adr-001.md
├── adr-002.md
├── api.md
├── incident-response.md
├── data-model.md
├── q3-launch-plan.md
├── superseded-design.md
└── onboarding.md
```

**Problems:**
- Architecture, ADRs, guides, and runbooks all live in one bucket — no separation of concerns
- Superseded docs (`superseded-design.md`) sit next to current docs — confusing
- A reader looking for "the runbook" has to scan everything
- Can't apply different rules (e.g., "runbooks need an owner; archive doesn't")

## Correct

```
✅ Purpose-based sub-folders
docs/
├── architecture/         # how the system is built (long-lived, slow-changing)
│   ├── overview.md
│   └── data-model.md
├── adr/                  # decisions made (append-only, numbered)
│   ├── 0001-record-architecture-decisions.md
│   └── 0002-choose-mysql-over-postgres.md
├── guides/               # how-to for developers (medium-lived, task-oriented)
│   ├── getting-started.md
│   ├── deployment.md
│   └── local-development.md
├── runbooks/             # ops procedures (short-titled, action-focused)
│   ├── deploy-production.md
│   └── incident-response.md
├── api/                  # API references (often generated; OpenAPI/Swagger)
│   └── openapi.yaml
└── archive/              # superseded but kept for history
    ├── 2024/
    └── 2025/
```

**Benefits:**
- Each folder has a clear purpose and audience
- Archive is visually separated from current docs
- Easy to apply per-folder rules (CODEOWNERS, freshness checks)
- Maps naturally to [Diátaxis](https://diataxis.fr/) categories (tutorials/how-to/reference/explanation)

## Diátaxis correspondence

| Sub-folder | Diátaxis | Audience |
|---|---|---|
| `guides/getting-started.md` | Tutorial | First-time users |
| `guides/deployment.md` | How-to | Engineers performing a task |
| `architecture/` | Explanation | Engineers building understanding |
| `api/` | Reference | Engineers looking up specifics |
| `adr/` | Decision record | Engineers asking "why?" |
| `runbooks/` | How-to (ops) | On-call engineers |

## Add folders as needed

- **`docs/security/`** — threat models, security architecture, audit reports
- **`docs/onboarding/`** — new-hire orientation, codebase tour
- **`docs/proposals/`** — RFCs / design proposals (before they become ADRs)
- **`docs/meeting-notes/`** — only if you'll actually maintain them; otherwise use the issue tracker

Don't pre-create empty folders. Add them when you have at least two docs that belong inside.

Reference: [Diátaxis](https://diataxis.fr/) · [adr.github.io](https://adr.github.io/)

---


## UPPERCASE Naming for Root-Level Conventional Files

**Impact: CRITICAL (Hosting platforms, tools, and readers expect specific filenames at root — case matters)**

GitHub, GitLab, Bitbucket, npm, Packagist, and most static-site generators look for **specific filenames in specific cases** at the repo root. `Readme.md` is not the same as `README.md` to a case-sensitive filesystem, and on case-sensitive CI runners (Linux containers) a mismatch breaks tools that auto-render the file.

## Incorrect

```
❌ Inconsistent / wrong casing
.
├── Readme.md                   # GitHub renders, but tools that grep "README" miss it
├── Changelog.md                # Keep-a-Changelog tooling expects CHANGELOG.md
├── license.md                  # Should be LICENSE (no extension by convention)
├── Contributing.md             # GitHub's "How to contribute" UI looks for CONTRIBUTING.md
└── security.md                 # GitHub security advisories look for SECURITY.md
```

**Problems:**
- GitHub case-sensitively matches `SECURITY.md` for the Security Advisories tab; `security.md` won't link
- Many CI tools (e.g., `markdownlint` rule MD041) and license detectors look up the exact uppercase name
- Linux filesystems treat `README.md` and `Readme.md` as distinct files — switching between editor casings creates phantom duplicates in git

## Correct

```
✅ Conventional UPPERCASE names at root
.
├── README.md                   # uppercase, .md extension
├── CHANGELOG.md
├── LICENSE                     # no .md extension (long-standing convention)
├── CONTRIBUTING.md
├── SECURITY.md
├── CODE_OF_CONDUCT.md          # underscores, not hyphens
└── AUTHORS.md                  # or MAINTAINERS.md
```

**Benefits:**
- GitHub auto-detects each file for its respective tab/feature (Security, Contributing, Code of Conduct)
- License-detection tools (GitHub Linguist, choosealicense.com) recognize `LICENSE` without extension
- Case-sensitive CI environments behave the same as your dev machine

## The `LICENSE` extension convention

`LICENSE` (no extension) is the long-standing convention from open-source culture. GitHub auto-detects the license type from the file content regardless of extension, but `LICENSE` without extension is the dominant pattern. `LICENSE.md` and `LICENSE.txt` also work; pick one and be consistent across your repos.

## Detection

```bash
# Find mis-cased conventional root files
ls *.md LICENSE* 2>/dev/null | \
  awk 'BEGIN{IGNORECASE=1}
       /^(readme|changelog|contributing|security|code[-_]of[-_]conduct|authors|maintainers)\.md$/ \
       && $0 !~ /^(README|CHANGELOG|CONTRIBUTING|SECURITY|CODE_OF_CONDUCT|AUTHORS|MAINTAINERS)\.md$/ \
       { print "MIS-CASED: " $0 }'

# Or simply:
ls Readme.md Changelog.md Contributing.md Security.md 2>/dev/null
```

Reference: [GitHub — Adding a security policy](https://docs.github.com/en/code-security/getting-started/adding-a-security-policy-to-your-repository) · [Keep a Changelog — naming](https://keepachangelog.com/)

---


## kebab-case for Files Under docs/

**Impact: CRITICAL (URLs, search, and grep all behave better with lowercase-hyphenated names)**

Files under `docs/` get linked, served as URLs by static-site generators, and grepped daily. `kebab-case` (lowercase, hyphen-separated) avoids case-sensitivity bugs, generates clean URLs (`docs/deployment-guide.md` → `/deployment-guide`), and is the dominant convention across documentation sites.

## Incorrect

```
❌ Mixed cases, spaces, underscores, PascalCase
docs/
├── DeploymentGuide.md          # PascalCase
├── deployment_guide.md         # snake_case
├── Deployment Guide.md         # spaces — break URLs
├── Architecture Overview.md    # spaces + Title Case
├── api-Reference.md            # mixed case
├── data_model.MD               # uppercase extension
└── Onboarding.md               # initial capital
```

**Problems:**
- Mixed conventions force readers to guess each filename
- Spaces in filenames produce `%20`-encoded URLs that are hard to type and look broken
- Case mismatches between branches cause "file not found" on Linux CI while working on macOS/Windows
- Static-site generators usually lowercase URLs anyway, so `DeploymentGuide.md` and `deployment-guide.md` collide

## Correct

```
✅ Consistent kebab-case throughout docs/
docs/
├── architecture/
│   ├── overview.md
│   ├── data-model.md
│   └── service-boundaries.md
├── guides/
│   ├── deployment-guide.md
│   ├── local-development.md
│   ├── getting-started.md
│   └── api-reference.md
└── runbooks/
    ├── incident-response.md
    └── deploy-production.md
```

**Benefits:**
- Predictable: anyone can guess the filename from the topic
- URL-friendly: `/docs/guides/deployment-guide` reads naturally
- Case-safe: lowercase eliminates case-sensitivity differences across OSes
- Greppable: `grep -rn 'deployment-guide' docs/` always finds the file

## Conventions

- **All lowercase** — `deployment-guide.md`, not `Deployment-Guide.md`
- **Hyphens, not underscores** — `data-model.md`, not `data_model.md`
- **No spaces** — ever
- **`.md` extension lowercase** — `.md`, not `.MD`
- **Keep names short** — 2–4 words ideal; if you need 6, the doc may be doing too much
- **Use nouns or noun-phrases** — `deployment-guide.md`, not `how-to-deploy.md` (the folder structure already conveys the verb)

## Detection

```bash
# Find files in docs/ with bad casing or characters
find docs/ -name '*.md' | grep -E '[A-Z]|[[:space:]]|_'
```

Any hit is a candidate for renaming via `git mv`. Add a markdownlint rule or a CI grep to prevent regression.

Reference: [Diátaxis — naming](https://diataxis.fr/) · [Markdownlint](https://github.com/DavidAnson/markdownlint)

---


## ADR File Naming — Numbered Prefix

**Impact: HIGH (ADRs are append-only history; numbering makes order, references, and freshness obvious)**

Architecture Decision Records (ADRs) are an append-only log of "why we chose X". A numbered prefix (`0001-...`, `0002-...`) makes chronology explicit, lets you say "see ADR-0007" in a PR, and groups all ADRs together when sorted alphabetically. Four-digit padding handles up to 9,999 ADRs without re-sorting.

## Incorrect

```
❌ Inconsistent or missing numbering
docs/adr/
├── record-architecture-decisions.md       # no number
├── 2-choose-mysql.md                      # single digit, sorts after 19
├── ADR-3-inertia-for-spa.md              # extra prefix
├── 0004-use-redis-cache.md
├── adopt-tailwind.md                      # no number
└── 10-restructure-services.md             # missing zero-padding
```

**Problems:**
- Listing `docs/adr/` sorts in unpredictable order (`10-...` comes before `2-...`)
- "See ADR 3" — but `ADR-3-inertia-for-spa.md` has the wrong prefix shape
- Some have numbers, some don't; you can't tell which were written first

## Correct

```
✅ Four-digit zero-padded prefix, kebab-case body
docs/adr/
├── 0001-record-architecture-decisions.md     # the meta-ADR — "we will record decisions"
├── 0002-choose-mysql-over-postgres.md
├── 0003-adopt-inertia-for-spa.md
├── 0004-use-redis-for-session-store.md
├── 0005-monorepo-package-layout.md
├── ...
└── 0042-rate-limit-public-api.md
```

**Benefits:**
- Lexicographic sort = chronological sort, always
- Compact references in PRs and code comments: "ADR-0007", "see #0007"
- Four digits handle a decade of decisions without renumbering
- New ADRs always append at the end — clear that history is immutable

## Naming components

```
0007-rate-limit-public-api.md
└┬─┘ └───────┬──────────────┘
 │           │
 │           └── Short description of the decision (kebab-case, 3–7 words)
 └── 4-digit zero-padded sequential number
```

- **Number** — sequentially assigned, never reused, never re-ordered
- **Description** — present-tense verb where applicable; matches the ADR title

## Template

Use [Michael Nygard's template](https://github.com/joelparkerhenderson/architecture-decision-record) (or `adr-tools` CLI). Each ADR has:

```markdown
# 0007. Rate-limit the public API

Date: 2026-05-16
Status: Accepted

## Context
What forces are at play?

## Decision
What did we decide?

## Consequences
What becomes easier? Harder?
```

## Tooling

```bash
# adr-tools CLI (https://github.com/npryce/adr-tools)
brew install adr-tools
adr init docs/adr
adr new "Rate-limit the public API"   # auto-creates 0007-rate-limit-the-public-api.md
```

## Status field — proposed → accepted → superseded

ADR status is part of the *content*, not the filename. Don't rename `0007-...` to `0007-SUPERSEDED-...`. Update the `Status:` line and link to the superseding ADR:

```markdown
Status: Superseded by ADR-0019
```

Reference: [adr.github.io](https://adr.github.io/) · [Michael Nygard's original post](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) · [adr-tools](https://github.com/npryce/adr-tools)

---


## Naming Anti-Patterns to Reject

**Impact: HIGH (Junk names accumulate fast — once you accept one, the floodgates open)**

A docs folder degrades one bad name at a time. `MyNotes.md` makes `JohnsThoughts.md` feel acceptable, and within a year you can't tell what's real documentation and what's someone's scratchpad. Reject these patterns at PR review time.

## The anti-patterns

### 1. Dates in filenames

```
❌ deployment-2025-09-14.md
❌ notes-2024-q3.md
❌ 2026-03-meeting.md
```

**Why bad:** dates make readers wonder which version is current. Use the `Last modified` git timestamp + a `Last verified:` line inside the doc instead.

**Allowed exception:** archive folders may include the year: `docs/archive/2024/launch-plan.md`.

### 2. First-person / owner names

```
❌ MyNotes.md
❌ JohnsArchitectureThoughts.md
❌ Asyraf-deployment-draft.md
```

**Why bad:** docs belong to the project, not a person. If only one person can maintain it, it's not documentation — it's a private note. Use the issue tracker or a personal scratchpad.

### 3. Draft / temp / version markers

```
❌ deployment-DRAFT.md
❌ architecture-FINAL.md
❌ architecture-FINAL-v2.md          (the FINAL-v2 paradox)
❌ deployment-OLD.md
❌ tmp-notes.md
❌ test-doc.md
```

**Why bad:** git history is the source of truth for "draft vs final" — that's what branches and PRs are for. "FINAL-v2" almost always means "we never deleted the old one".

### 4. Mixed-purpose / vague names

```
❌ misc.md
❌ stuff.md
❌ notes.md (at root)
❌ documentation.md (the whole project's docs in one file)
❌ general.md
```

**Why bad:** if you can't name the doc precisely, it doesn't have a clear purpose. Either split it into focused docs or delete it.

### 5. AI-plan / status / summary files

```
❌ PLAN.md
❌ IMPLEMENTATION-PLAN.md
❌ REFACTOR-PLAN.md
❌ IMPLEMENTATION-SUMMARY.md
❌ COMPLETED.md
❌ NEXT-STEPS.md
❌ TODO.md
```

**Why bad:** these are agent-generated transient state, not documentation. The work either landed (history is in git/PRs) or it didn't (tracking belongs in the issue tracker). See `cleanup-ai-junk`.

### 6. Numbered without ADR semantics

```
❌ doc-1.md, doc-2.md, doc-3.md         (numbers without meaning)
❌ chapter-1.md, chapter-2.md            (this is a book, not a docs folder)
```

**Why bad:** numbering implies order, but these have no append-only / decision-record semantics. Either use ADRs (`docs/adr/0001-...`) or use descriptive names.

## Correct — what to use instead

| Bad | Good |
|---|---|
| `MyArchitectureNotes.md` | `docs/architecture/overview.md` |
| `deployment-2025-09.md` | `docs/guides/deployment.md` (with internal `Last verified:` line) |
| `deployment-DRAFT.md` | Open a PR; keep the draft on a branch |
| `notes.md` | Either a focused doc OR delete and use the issue tracker |
| `PLAN.md` | A linked GitHub issue or project board |
| `architecture-FINAL-v2.md` | `docs/architecture/overview.md` + git history |

## Detection

```bash
# Names containing dates, draft markers, first-person, or AI-junk patterns
find docs/ -name '*.md' | grep -Ein \
  '/(DRAFT|FINAL|OLD|TMP|TEMP|TODO|PLAN|SUMMARY|NEXT-STEPS|My[A-Z][a-z]|[12][0-9]{3}-[0-9]{2}-[0-9]{2}|v[0-9]+\.md)'
```

Reference: [Diátaxis](https://diataxis.fr/) · [Documentation System — naming conventions](https://docs.divio.com/documentation-system/)

---


## README — Required Content

**Impact: CRITICAL (First impression for every reader; missing or stale README is the #1 onboarding blocker)**

The README is the front door. Every new contributor, every dependency review, every "is this project maintained?" check starts here. A weak README means each onboarding repeats the same questions in Slack.

## Required sections (in order)

````markdown
# Project Name

One-sentence description: what does this do, for whom.

[Badges optional — build, version, license]

## Overview

2–3 sentences expanding on the one-liner. Who uses this, what problem it solves.

## Requirements

- PHP 8.3+ / Node 22+
- MySQL 8.0+ / Redis (if used)
- Any other prerequisites

## Installation

```bash
git clone …
composer install
npm install
cp .env.example .env
php artisan key:generate
php artisan migrate
```

## Usage

How to run it locally:

```bash
php artisan serve            # backend
npm run dev                  # frontend
```

How to run tests:

```bash
php artisan test
npm test
```

## Documentation

- [Architecture overview](docs/architecture/overview.md)
- [Deployment guide](docs/guides/deployment.md)
- [API reference](docs/api/openapi.yaml)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

[MIT](LICENSE) (or whichever)
````

## Incorrect

```markdown
❌ Sparse README missing essentials
# my-app

A web app.

## Setup

Run npm install. Should work.
```

**Problems:**
- No description of *what* the app does or *for whom*
- No requirements (which Node version? what database?)
- "Should work" — when it doesn't, the reader has no context
- No path to architecture docs, deployment, contributing
- No license — many companies' legal review will block adoption

## What to include vs leave out

**Include:**
- One-sentence description
- Requirements (with versions)
- Install / run / test commands
- Links to deeper docs (`docs/`)
- License

**Don't include:**
- A full API reference (link to `docs/api/`)
- Detailed architecture (link to `docs/architecture/overview.md`)
- Release history (that's `CHANGELOG.md`)
- Long FAQs (`docs/guides/troubleshooting.md`)

A README that's longer than one screen is doing too much. Split it.

## Verification

Add a `Last verified: YYYY-MM-DD` line at the top of installation. Once a quarter, walk through the steps from scratch on a clean checkout and update the date.

```markdown
## Installation

_Last verified on a clean checkout: 2026-04-12_

git clone …
…
```

## Detection

```bash
# Does README exist and have minimum required sections?
test -f README.md || echo "MISSING README.md"
for h in 'Overview' 'Requirements' 'Installation' 'Usage' 'License'; do
  grep -q "^## $h" README.md || echo "MISSING SECTION: ## $h"
done
```

Reference: [Make a README](https://www.makeareadme.com/) · [GitHub — About READMEs](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes)

---


## CHANGELOG — Keep-a-Changelog Format

**Impact: HIGH (Release history readers can actually parse; the alternative is reading every commit)**

Every project past its first release needs a CHANGELOG that humans can read in 30 seconds. "What changed in v3.2?" should not require `git log v3.1..v3.2`. The [Keep a Changelog](https://keepachangelog.com/) format is the de-facto standard: one section per version, grouped by change type, newest first.

## Required structure

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- Bullet for each new feature in flight

## [1.3.0] - 2026-05-10

### Added
- Bulk user export endpoint (#412)
- MySQL 8.4 support; bumped CI to test against it

### Changed
- Password hashing default switched to argon2id (existing bcrypt hashes still validated and re-hashed on login)

### Fixed
- Stripe webhook idempotency key handling (#398, #401)

### Security
- Bumped guzzlehttp/guzzle to 7.9.2 (CVE-2024-XXXX)

## [1.2.1] - 2026-04-22

### Fixed
- Cursor pagination skipping the last row of each page (#387)

## [1.2.0] - 2026-04-01

### Added
- Initial release of public REST API

[Unreleased]: https://github.com/org/repo/compare/v1.3.0...HEAD
[1.3.0]: https://github.com/org/repo/compare/v1.2.1...v1.3.0
[1.2.1]: https://github.com/org/repo/compare/v1.2.0...v1.2.1
[1.2.0]: https://github.com/org/repo/releases/tag/v1.2.0
```

## Change-type buckets (use these)

| Bucket | When to use |
|---|---|
| **Added** | New features or capabilities |
| **Changed** | Changes to existing functionality |
| **Deprecated** | Soon-to-be-removed features (announce here, then remove next major) |
| **Removed** | Removed features |
| **Fixed** | Bug fixes |
| **Security** | CVE patches, vulnerability fixes |

Drop unused buckets from a given release (don't include empty `### Added` if there were no additions).

## Incorrect

```markdown
❌ Free-form CHANGELOG that nobody can scan

# Changes

v1.3 — May 2026 — lots of stuff, see commits for details
v1.2 — April 2026 — bugfixes and improvements
v1.1 — March 2026 — added cool feature
v1.0 — Feb 2026 — initial release
```

**Problems:**
- "Lots of stuff" tells the reader nothing
- No buckets, so security fixes mix with cosmetic changes
- No dates in ISO format (international ambiguity)
- No links to commits, PRs, or version tags

## CHANGELOG-in-the-PR discipline

A CHANGELOG entry should land in **the same PR** as the change it describes — not after the fact, not "later in a batch". This is the only way to keep it accurate.

Enforce with CI:

```yaml
# .github/workflows/changelog.yml
- name: CHANGELOG entry required for non-trivial changes
  run: |
    if git diff --name-only origin/main...HEAD | grep -qvE '^(\.github/|docs/|tests?/|README\.md|\.gitignore)$'; then
      git diff origin/main...HEAD -- CHANGELOG.md | grep -q '^+' \
        || { echo "Add a CHANGELOG.md entry"; exit 1; }
    fi
```

Skip for docs-only, test-only, and trivial PRs (use a label or PR title prefix).

## When to start a CHANGELOG

- **Library / package** — from the very first published version
- **Application** — from the first deployment to production OR the first release tagged with `v`
- **Internal tool** — when more than one team consumes it

For pre-1.0 projects in flux, [Keep a Changelog](https://keepachangelog.com/en/1.1.0/#how) explicitly endorses starting with `0.x.y` and using `[Unreleased]` heavily.

Reference: [Keep a Changelog](https://keepachangelog.com/) · [Semantic Versioning](https://semver.org/)

---


## LICENSE — Required Legal Clarity

**Impact: CRITICAL (Without a license, code is technically un-reusable; many companies' legal review blocks adoption outright)**

A repository without a LICENSE file means **all rights reserved** by default in most jurisdictions — nobody can legally use, copy, or modify the code, even if it's public. Every serious open-source project has one, and most companies' legal teams reject internal adoption of unlicensed dependencies.

## Required

```
LICENSE                       # at repo root, no extension by convention
```

Contents: the full text of the license you've chosen. For MIT, that's the standard ~20-line text with `<year>` and `<copyright holder>` filled in.

## Choosing a license

Use [choosealicense.com](https://choosealicense.com/) — it asks 2–3 questions and recommends one. Common picks:

| License | When | Notes |
|---|---|---|
| **MIT** | Permissive; you want maximum adoption | Short, well-known, allows commercial use |
| **Apache-2.0** | Permissive + explicit patent grant | Slightly longer; common for company-backed OSS |
| **AGPL-3.0** | Copyleft + network-use clause | SaaS deployments must share modifications |
| **BSL** / **Elastic 2.0** | Source-available, time-delayed open | "Open-ish" — verify it suits your goal |
| **Proprietary** | Closed-source | Add `LICENSE` saying "All rights reserved" + contact for commercial terms |

For internal-only repos, still include a `LICENSE` file stating the company holds copyright and the code is for internal use only. This removes ambiguity for departing employees and contractors.

## Incorrect

```
❌ No LICENSE file at all
.
├── README.md
├── CHANGELOG.md
└── src/...
```

```
❌ Wrong placement / naming
.
├── docs/license.txt          (should be at root, named LICENSE)
└── LICENSE.MD                (works but unconventional case)
```

```
❌ "MIT" mentioned only in README, no file
README:
> ## License
> MIT
```

(GitHub will not detect this as MIT licensed; license badges and tooling will fail.)

## Correct

```
✅ LICENSE at root
.
├── LICENSE                   ← the full license text
├── README.md
└── src/...
```

README links to it:

```markdown
## License

[MIT](LICENSE)
```

## License consistency in package manifests

Keep the manifest field in sync with the LICENSE file:

```json
// package.json
{
  "license": "MIT"
}
```

```json
// composer.json
{
  "license": "MIT"
}
```

Mismatch (LICENSE says MIT, `package.json` says Apache-2.0) trips up `npm audit`, license-scanning tools, and human reviewers.

## Detection

```bash
test -f LICENSE -o -f LICENSE.md -o -f LICENSE.txt || echo "MISSING LICENSE"

# Compare LICENSE to package.json (rough sanity check)
LICENSE_NAME=$(grep -oE '(MIT|Apache|BSD|GPL|ISC|MPL)' LICENSE | head -1)
PKG_LICENSE=$(node -e "console.log(require('./package.json').license)" 2>/dev/null)
echo "LICENSE file: $LICENSE_NAME, package.json: $PKG_LICENSE"
```

Reference: [Choose a License](https://choosealicense.com/) · [GitHub — Licensing a repository](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/licensing-a-repository) · [SPDX license list](https://spdx.org/licenses/)

---


## CONTRIBUTING — When and What

**Impact: MEDIUM (Sets the rules of engagement; without it, every PR is a guess)**

A `CONTRIBUTING.md` tells outside (or new-to-the-team) contributors how to participate: which branch to target, commit message style, test expectations, and how PRs get reviewed. GitHub auto-links to it whenever someone opens an issue or PR.

## When required

- **Open-source projects** — required from day one
- **Internal projects with external collaborators** (consultants, contractors) — recommended
- **Solo or single-team internal projects** — optional; an `.md` heading inside the README's "Development" section is fine for small teams

If the project is closed-source and only your team contributes, skip `CONTRIBUTING.md` and put the same information in the README.

## Recommended structure

```markdown
# Contributing

Thanks for considering a contribution!

## Branching

- `main` is protected; PRs only
- Feature branches: `feat/<short-desc>`
- Bug fixes: `fix/<short-desc>`

## Commit messages

We follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat(auth): add MFA enrollment endpoint
fix(checkout): handle expired payment intent
docs(readme): clarify install steps
```

## Development setup

```bash
git clone …
composer install && npm install
cp .env.example .env
php artisan migrate
```

## Running tests

```bash
php artisan test         # PHP / Laravel
npm test                 # JS / TS
```

All tests must pass; new features need new tests.

## Pull request checklist

- [ ] CHANGELOG.md updated (under `[Unreleased]`)
- [ ] Tests added / updated
- [ ] `composer.lock` / `package-lock.json` committed if deps changed
- [ ] Linked to a GitHub issue (if applicable)

## Review process

- One maintainer approval required
- CI must pass (`tests`, `lint`, `markdown`)
- Squash-merge — keep `main` history linear

## Code of Conduct

This project follows the [Contributor Covenant](CODE_OF_CONDUCT.md).
```

## Incorrect

```markdown
❌ Vague CONTRIBUTING that says "just open a PR"

# Contributing

Open a PR. We'll review.
```

**Problems:**
- No branch policy → reviewer has to comment on every PR
- No test expectation → contributors merge without running them
- No commit style → CHANGELOG generation is harder, history is noisy

## Detection

```bash
# Does CONTRIBUTING exist where needed?
if [ -f "package.json" ] && grep -q '"private"' package.json; then
  : # internal, optional
else
  test -f CONTRIBUTING.md || echo "MISSING CONTRIBUTING.md (recommended for non-private projects)"
fi
```

Reference: [GitHub — Setting guidelines for repository contributors](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/setting-guidelines-for-repository-contributors) · [Conventional Commits](https://www.conventionalcommits.org/)

---


## SECURITY.md — Vulnerability Reporting Policy

**Impact: HIGH (Without a clear policy, researchers go public; with one, they email you privately)**

A `SECURITY.md` tells security researchers how to report a vulnerability to you privately. Without it, well-intentioned researchers either open a public GitHub issue (instant CVE) or give up and never report. GitHub's "Security" tab links directly to this file.

## When required

- **Any internet-facing service** — required
- **Open-source libraries** — required from first public release
- **Internal-only projects** — optional, but useful for clarity even internally
- **Closed-source applications used by customers** — recommended

## Recommended structure

```markdown
# Security Policy

## Supported versions

We provide security updates for the following versions:

| Version | Supported |
|---------|-----------|
| 3.x     | ✅        |
| 2.x     | ✅ (until 2026-12) |
| 1.x     | ❌ (EOL)  |

## Reporting a vulnerability

**Please do not file a public GitHub issue for security vulnerabilities.**

Email us at **security@example.com** with:
- A description of the issue
- Steps to reproduce
- Affected versions (if known)
- Any proof-of-concept code (optional)

We acknowledge reports within **2 business days** and aim to issue a fix within **30 days** for high/critical severity.

If you prefer encrypted communication, our PGP key is at <https://example.com/security.asc>.

## Disclosure timeline

- Day 0: We receive your report and acknowledge
- Day 1–14: Triage, reproduce, develop fix
- Day 15–30: Release patched version, notify users
- Day 30+: Public disclosure (we'll credit you unless you prefer anonymity)

## Safe harbor

We will not pursue legal action against researchers who:
- Make a good-faith effort to follow this policy
- Avoid privacy violations, service disruption, or destruction of data
- Give us reasonable time to remediate before public disclosure
```

## Incorrect

```markdown
❌ No SECURITY.md at all on a public web app

(No mechanism for reporting; researchers either guess an email or go public.)
```

```markdown
❌ "Just open an issue"

# Security

If you find a security issue, please open a GitHub issue.
```

(This guarantees public disclosure of vulnerabilities — the worst possible default.)

## Use GitHub's built-in private reporting

GitHub supports **Private Security Advisories** — researchers can report via the Security tab without exposing the issue publicly. Enable in Settings → Security → "Private vulnerability reporting". Reference it in `SECURITY.md`:

```markdown
You may also use **GitHub's private vulnerability reporting**:
<https://github.com/org/repo/security/advisories/new>
```

## Detection

```bash
# Internet-facing? Public repo? — should have SECURITY.md
test -f SECURITY.md || test -f .github/SECURITY.md \
  || echo "MISSING SECURITY.md (required for public/internet-facing projects)"
```

Reference: [GitHub — Adding a security policy](https://docs.github.com/en/code-security/getting-started/adding-a-security-policy-to-your-repository) · [GitHub Private Vulnerability Reporting](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing-information-about-vulnerabilities/privately-reporting-a-security-vulnerability) · [Disclose.io safe-harbor template](https://github.com/disclose/disclose)

---


## Conciseness — Cut Bloat

**Impact: HIGH (Long docs aren't more informative — they're more skipped. The real risk is signal-to-noise.)**

A README that's 400 lines doesn't tell the reader more — it tells them less, because they stop reading after the first screen. Good docs say what's needed and stop. Length is a cost, not a virtue.

## Bloat patterns to cut

### 1. Opening boilerplate

```markdown
❌ # Deployment Guide

   This document describes the deployment process for our application.
   It is intended to be read by engineers who are responsible for
   deploying the application to production. Before reading this document,
   you should be familiar with the basics of the application architecture.

   Without further ado, let's dive into the deployment process.

   ## Overview
   ...
```

```markdown
✅ # Deployment Guide

   ## Overview
   ...
```

The reader knows it's a deployment guide — the H1 says so. They know it's for engineers — they're reading the engineering docs. Five paragraphs of "this document describes" is five paragraphs of throat-clearing.

### 2. Restating what code or commands already say

````markdown
❌ ## Installation

   To install the application, you first need to clone the repository
   from GitHub. After cloning the repository, navigate into the
   newly-created directory. Then, install the PHP dependencies by
   running composer install. Once that completes, install the
   JavaScript dependencies by running npm install. Finally, copy the
   example environment file to a new .env file.

   ```bash
   git clone …
   cd …
   composer install
   npm install
   cp .env.example .env
   ```
````

````markdown
✅ ## Installation

   ```bash
   git clone …
   cd …
   composer install
   npm install
   cp .env.example .env
   ```
````

The commands are self-explanatory. Don't narrate them.

### 3. Repetition across sections

A README that says the same thing in the "Overview", then again in "Description", then again in "About this project". Pick one.

### 4. "As mentioned above" / "as we'll see later"

If something needs cross-referencing, link to it. Inline meta-commentary about the document's structure is noise.

### 5. Over-explaining basics

```markdown
❌ Run `composer install`. Composer is a dependency manager for PHP
   that allows you to declare the libraries your project depends on
   and manages (install/update) them for you. To learn more about
   Composer, visit https://getcomposer.org.
```

If the reader is on the install page of your project, they know what Composer is (or they have one click to learn). Don't teach the basics inline.

## Rough length targets

These are heuristics — focused docs can be longer if they have to be, and short docs can be too long if they're padded.

| Doc | Target |
|---|---|
| README | < 200 lines (link to deeper docs for detail) |
| Single guide / runbook | < 300 lines |
| Single ADR | < 150 lines (decision + context + consequences) |
| Architecture overview | < 500 lines (split into multiple docs if longer) |

If you hit these and the content is genuinely necessary, split into multiple focused docs rather than letting one file balloon.

## The "delete a paragraph" exercise

Before publishing a doc, try this: pick any paragraph and ask, "if I deleted this, would a reader miss anything important?" If the answer is no — or "they'd just need to read the next paragraph more carefully" — delete it.

Most docs improve after a 20–30% cut.

## Detection

```bash
# Long markdown files — review for bloat
find docs/ -name '*.md' -exec wc -l {} \; | sort -rn | head -20

# Docs with low signal: many lines containing "this document", "you should", "let's"
# Note: grep -c counts matching LINES (one filler line = 1, even with multiple phrases).
grep -rEcH "(this document|you should|let'?s (dive|look|start))" --include='*.md' docs/ \
  | awk -F: '{ if ($2 > 5) print "BLOATY: " $1 " (" $2 " matching lines)" }'
```

Reference: [Strunk & White — Omit Needless Words](https://en.wikipedia.org/wiki/The_Elements_of_Style) · [Diátaxis — "less is more"](https://diataxis.fr/) · Internal: [`quality-ai-slop`](quality-ai-slop.md) (AI-style filler patterns)

---


## AI Slop in Documentation Content

**Impact: HIGH (AI-generated content has telltale fingerprints that erode docs quality file-by-file)**

[`cleanup-ai-junk`](cleanup-ai-junk.md) covers whole files agents leave behind. This rule covers **content inside otherwise-legitimate docs** — a README, an architecture overview, a runbook — that has been bulked up with AI-style filler. The patterns are recognizable; once you see them, you can't unsee them.

## Slop fingerprints in content

### 1. Filler phrases — delete on sight

```
"It's worth noting that..."
"It's important to understand..."
"In summary..."  /  "To summarize..."
"Let's dive into..."  /  "Let's take a look at..."
"As we'll see..."  /  "As mentioned earlier..."
"That being said..."  /  "Having said that..."
"At its core..."  /  "Essentially..."
"In essence..."  /  "Fundamentally..."
"It goes without saying..."
"Needless to say..." (if it's needless to say, don't say it)
```

These phrases add words without adding meaning. A human technical writer doesn't sprinkle "in essence" through a deployment guide.

### 2. Bullet-list explosions

```markdown
❌ ## Why we chose Redis

   Redis was chosen for the following reasons:

   - **Performance** — Redis is very fast
   - **Reliability** — Redis is highly reliable
   - **Scalability** — Redis can scale to large workloads
   - **Community** — Redis has a large and active community
   - **Documentation** — Redis is well-documented
   - **Ecosystem** — Redis has a rich ecosystem of tools
   - **Maturity** — Redis is a mature and proven technology
   - **Versatility** — Redis supports many use cases
```

Eight bullets, zero specifics. Replace with one paragraph that says *what we actually needed* (sub-millisecond GET latency + pub-sub for session invalidation) and *why Redis fit* (battle-tested at our scale; team has prod experience). The version with bullets *looks* informative; the prose version *is* informative.

### 3. Closing sign-offs

```markdown
❌ ## Conclusion

   I hope this guide has been helpful in setting up your development
   environment. If you have any questions, feel free to reach out to
   the team. Happy coding! 🎉
```

Documentation isn't a blog post. Cut the goodbye.

### 4. Past-tense narration where present tense fits

```markdown
❌ When we started this project, we considered using Laravel because
   it offered a good balance of features. We decided that Laravel was
   the right choice. We chose MySQL as our database because of its
   maturity.
```

```markdown
✅ The application uses Laravel for its mature ecosystem and MySQL
   for its operational track record. See [ADR-0002](adr/0002-...) for
   the full decision context.
```

The "we did X" narrative belongs in commit messages or ADRs, not in the current architecture doc.

### 5. Generic praise / hedging

```
"leveraging industry best practices"
"following modern development standards"
"a robust and scalable solution"
"a powerful and flexible framework"
"state-of-the-art technology"
```

These phrases are zero-content. If "industry best practices" matters, name the specific practice (e.g., "OWASP ASVS Level 2", "12-factor app", "Conventional Commits"). If "scalable" matters, name the actual numbers (e.g., "10k req/s with p95 < 200ms").

### 6. Markdown formatting overuse

- **Random words bolded for no reason**
- Excessive heading nesting (`####` after only one `##`)
- Bullet lists where prose flows better
- Tables for two items

### 7. The 100-line answer to a 3-line question

A "How do I run tests?" section that takes 100 lines to say `php artisan test`.

## Detection

```bash
# Filler-phrase fingerprints
grep -rEi --include='*.md' \
  "it's worth noting|let's (dive|take a look|start)|it goes without saying|needless to say|that being said|having said that|at its core|in essence|fundamentally" \
  docs/ README.md

# Closing sign-offs in docs
grep -rEi --include='*.md' \
  "happy coding|hope this (was )?help|if you have any questions|feel free to reach out" \
  docs/ README.md

# Generic praise (likely AI-slop)
grep -rEi --include='*.md' \
  "industry best practices|state-of-the-art|robust and scalable|powerful and flexible" \
  docs/ README.md

# Many emojis = often AI-generated (humans rarely emoji-pepper technical docs).
# `grep -roE` prints one match per emoji occurrence (not per line), so the
# per-file count below is the true emoji count.
find docs/ README.md -name '*.md' -type f 2>/dev/null | while read f; do
  N=$(grep -oE '🤖|✨|🚀|🎉|💡|⚡|📝|🔥|🎯|💪' "$f" | wc -l | tr -d ' ')
  if [ "$N" -gt 5 ]; then
    echo "MANY EMOJIS ($N): $f"
  fi
done
```

## How to triage a slop-flagged doc

1. **Read the whole doc end to end** — is the content valid under the slop?
2. **If valid** — rewrite the prose: cut filler, name specifics, prefer present tense
3. **If hollow** — the slop was hiding that the doc has no real content. Either rewrite from scratch with real content, or delete.

**Never** rewrite docs *with another LLM* without close review — you'll just produce different slop. Use the model to draft, then read every line as a human and cut ruthlessly.

Reference: Internal: [`cleanup-ai-junk`](cleanup-ai-junk.md) (junk files) · [`quality-conciseness`](quality-conciseness.md) (general bloat) · [On Bullshit (Frankfurt)](https://en.wikipedia.org/wiki/On_Bullshit) — yes, really

---


## Heading Hierarchy — H1 Once, No Skipped Levels

**Impact: HIGH (Broken heading hierarchy breaks readability, accessibility, and auto-generated TOCs)**

Each markdown file has exactly one H1 (the document title), and lower levels go in sequence (H2 → H3 → H4) without skipping. Auto-generated TOCs, screen readers, and GitHub's outline view all rely on this. A file with three H1s or with H1 → H3 jumps reads as broken to humans and as malformed to tools.

## Rules

1. **Exactly one H1 per file** — it's the title
2. **No skipping levels** — H2 can be followed by H2 or H3, but not H4
3. **Don't use bold instead of a heading** — `**Important:**` doesn't show up in TOCs
4. **Don't use H1 inside a doc** — once you've used `#`, use only `##` and below
5. **Headings should be descriptive, not generic** — `## Configuration` not `## Section 2`

## Incorrect

### Multiple H1s

```markdown
# Deployment Guide

## Overview

Some overview content.

# Configuration

Some configuration content.       ← second H1; should be ##

# Troubleshooting

…                                  ← third H1
```

The doc has three "top-level" sections in markdown's eyes; GitHub will treat each H1 as a candidate document title. The TOC will be flat and confused.

### Skipped levels

```markdown
# Architecture Overview

## Components                       ← H2

#### Authentication                 ← H4 (skipped H3)

Some content about auth.

#### Authorization                  ← H4 still

#### Sessions                       ← H4 still
```

The reader expects "Components" to have direct sub-sections; instead it has sub-sub-sections. Auto-generated outlines look broken.

### Bold-as-heading

````markdown
## Setup

**Prerequisites:**                  ← bold, not heading

- Node 22
- MySQL 8

**Install:**                        ← bold, not heading

```bash
npm install
```
````

"Prerequisites" and "Install" don't appear in the TOC; readers scanning headings miss them.

## Correct

```markdown
# Deployment Guide                  ← exactly one H1, matching the file's purpose

## Overview

Brief overview content.

## Configuration

### Environment variables           ← H3 under H2

### Secrets storage

## Troubleshooting

### Build failures                  ← H3 under H2

### Deploy timeouts
```

Sequential, predictable, scannable.

## Heading content

### Use sentence case

```
✓ ## Setting up a development environment
✗ ## Setting Up A Development Environment   (title case is awkward to read)
✗ ## SETTING UP A DEVELOPMENT ENVIRONMENT   (shouting)
```

### Be specific

```
✓ ## Resetting a user's password from the admin panel
✗ ## Password reset                          (which kind? from where?)
```

### Match how readers search

The TOC of a 1000-line doc is its index. Headings should answer "what would I search for?", not be cute.

## Detection

```bash
# Files with more than one H1 (set -e safe — uses if/then instead of && chain)
find docs/ -name '*.md' -print0 | while IFS= read -r -d '' f; do
  H1_COUNT=$(grep -cE '^# [^#]' "$f" || true)
  if [ "$H1_COUNT" -gt 1 ]; then
    echo "MULTIPLE H1s ($H1_COUNT): $f"
  fi
done

# Skipped heading levels (markdownlint rule MD001)
npx markdownlint-cli2 'docs/**/*.md' 'README.md'
# (Configure .markdownlint.json: { "MD001": true })
```

`markdownlint`'s rule MD001 (`heading-increment`) catches skipped levels automatically; MD025 (`single-h1`) catches multiple H1s. Enable both.

Reference: [Markdownlint rules MD001 / MD025](https://github.com/DavidAnson/markdownlint/blob/main/doc/Rules.md) · [WAI-ARIA Heading Levels](https://www.w3.org/WAI/tutorials/page-structure/headings/)

---


## Code Blocks — Language Tags and Copy-Pasteability

**Impact: MEDIUM (Untagged code blocks lose syntax highlighting; broken commands waste reader time)**

A code block without a language tag renders as monospace text with no syntax highlighting. A "command" that contains an unexplained placeholder (`<your-token-here>`) or a typo wastes every future reader's time. Code blocks in docs are contracts: if you paste them, they should work.

## Rules

### 1. Every code block has a language tag

````markdown
❌ ```
   php artisan migrate
   ```

✅ ```bash
   php artisan migrate
   ```
````

Common tags:

| Tag | Use for |
|---|---|
| `bash` / `sh` | Shell commands, terminals |
| `php` | PHP code |
| `js` / `ts` / `tsx` | JavaScript / TypeScript / TSX |
| `json` | JSON config |
| `yaml` / `yml` | YAML (CI configs, etc.) |
| `sql` | SQL queries |
| `markdown` / `md` | Nested markdown examples (use 4-backtick outer fence) |
| `diff` | Patches / before-after |
| `text` or no tag | Genuinely plain text only |

### 2. Commands you intend to be copy-pasted must actually run

```bash
❌ git clone <your-repo-url>            # placeholder; reader has to figure out what
❌ npm install your-package             # 'your-package' isn't a real package

✅ git clone git@github.com:your-org/your-repo.git
✅ npm install                           # no args — installs from package.json
```

If a placeholder is unavoidable (real credentials, secret URL), surround it with a comment that makes the substitution obvious:

```bash
✅ # Replace YOUR_API_TOKEN with the token from Settings → API
   curl -H "Authorization: Bearer YOUR_API_TOKEN" https://api.example.com/...
```

### 3. Multi-line commands use proper line continuation

```bash
❌ git commit -m "feat: add user export"
   --no-verify

✅ git commit -m "feat: add user export" \
              --no-verify
```

Without the backslash, the second line is a separate command (and will fail).

### 4. Don't include `$` or `>` prompts in copy-pasteable blocks

```bash
❌ $ npm install
   $ npm run dev

✅ npm install
   npm run dev
```

The `$` is fine if you're showing input/output together (where output lines have no `$`), but for copy-paste-friendly blocks, omit the prompt.

### 5. Show output separately from input

````markdown
❌ ```bash
   $ php artisan about
   Laravel ............. 11.0
   PHP ................. 8.3
   ```

✅ ```bash
   php artisan about
   ```

   Output:

   ```
   Laravel ............. 11.0
   PHP ................. 8.3
   ```
````

This way the reader can copy the command without dragging output along.

### 6. Test the commands

Before publishing a guide, run every command in it from scratch (clean shell, clean checkout). The number of bugs you find on the first run is sobering.

## Detection

```bash
# Code blocks with no language tag
for f in $(find docs/ README.md -name '*.md' 2>/dev/null); do
  awk -v file="$f" '
    /^```$/ && !in_code { print file ":" NR ": untagged code block"; in_code=1; next }
    /^```/  && !in_code { in_code=1; next }
    /^```$/ &&  in_code { in_code=0 }
  ' "$f"
done

# Markdownlint rule MD040 (fenced-code-language) catches this automatically
npx markdownlint-cli2 --config '.markdownlint.json' '**/*.md'
```

Add to `.markdownlint.json`:

```json
{
  "MD040": true        // fenced-code-language — language tags required
}
```

Reference: [Markdownlint MD040](https://github.com/DavidAnson/markdownlint/blob/main/doc/Rules.md#md040) · [GitHub — Syntax highlighting](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-and-highlighting-code-blocks)

---


## Links — Descriptive Text and No Broken URLs

**Impact: MEDIUM ("Click here" is unscannable and inaccessible; broken links erode trust in the whole doc set)**

Two failure modes degrade docs quickly: link text that doesn't describe where the link goes ("click here", "this link") and links that 404 because the target moved. Both are easy to detect and fix; left unchecked, both compound until readers stop trusting the docs.

## Rule 1 — Descriptive link text

```markdown
❌ For deployment instructions, click [here](docs/guides/deployment.md).
❌ See [this page](docs/guides/deployment.md) for deployment.
❌ Read more about deployment [here](docs/guides/deployment.md).

✅ See the [deployment guide](docs/guides/deployment.md).
✅ Deployment instructions are in [docs/guides/deployment.md](docs/guides/deployment.md).
✅ For details, see the [deployment guide](docs/guides/deployment.md).
```

Why descriptive text matters:

1. **Scannability** — a reader skimming the doc sees "deployment guide", not "click here"
2. **Accessibility** — screen readers can announce just the link text in isolation; "click here" out of context is meaningless
3. **Search** — "deployment guide" is a useful keyword; "click here" is not

### Quick test

Read just the link text out loud (ignore the surrounding sentence). Does it tell you where you'd land? If yes, it's good. If "click here" / "this link" / "more", rewrite.

## Rule 2 — Use relative paths for in-repo links

```markdown
❌ See https://github.com/your-org/your-repo/blob/main/docs/architecture/overview.md
✅ See [docs/architecture/overview.md](docs/architecture/overview.md)
```

Relative paths:
- Work on any fork or mirror
- Don't break when the repo is renamed or moved
- Resolve correctly when docs are served via MkDocs / GitHub Pages

Use absolute URLs only for **off-repo** targets (external sites, other repos).

## Rule 3 — Check links don't 404

Broken links happen when files are renamed, moved, or deleted without updating their referrers. The fix is automation: a link checker in CI.

```yaml
# .github/workflows/check-links.yml
name: Check links
on:
  pull_request:
    paths: ['**/*.md']
  schedule: [{ cron: '0 9 * * MON' }]   # weekly catches link-rot

jobs:
  lychee:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: lycheeverse/lychee-action@v2
        with:
          args: --no-progress --exclude-mail './**/*.md' README.md
          fail: true
```

[Lychee](https://github.com/lycheeverse/lychee) is the recommended tool: fast (Rust), supports config files, exits non-zero on broken links so CI fails the PR.

For per-file checks during local development:

```bash
npx markdown-link-check docs/guides/deployment.md
```

## Rule 4 — Don't link to the current file

```markdown
❌ See [this section](#installation) for installation.   (when "this section" IS the installation section)
```

Self-references are usually filler. The reader is already there.

## Rule 5 — Use reference-style links for repeated targets

When the same URL appears 3+ times in a doc, use reference-style:

```markdown
Some prose linking to the [Laravel docs][laravel].

Some more prose also linking to the [Laravel docs][laravel] later on.

Yet more prose, third time linking to the [Laravel docs][laravel].

[laravel]: https://laravel.com/docs/11.x
```

The doc stays readable in source form, and updating the URL in one place updates all occurrences.

## Detection

```bash
# Non-descriptive link text
grep -rEnH '\[(click here|here|this|this link|read more|more|link)\]\(' \
  --include='*.md' docs/ README.md

# Absolute URLs to your own repo (should be relative)
# EDIT: replace with your actual GitHub org/repo before running.
ORG_REPO="your-org/your-repo"
grep -rEnH "github\.com/$ORG_REPO/(blob|tree)/" --include='*.md' .

# Run link checker locally
npx lychee --no-progress './**/*.md' README.md
```

Reference: [Lychee](https://github.com/lycheeverse/lychee) · [markdown-link-check](https://github.com/tcort/markdown-link-check) · [WCAG 2.4.4 — Link Purpose](https://www.w3.org/WAI/WCAG21/Understanding/link-purpose-in-context.html)

---


## AI-Generated Junk Files

**Impact: HIGH (Agents create transient plan/summary files that masquerade as documentation)**

When you let coding agents work on a project, they tend to leave behind plan files, status summaries, and "next steps" notes that look like documentation but are actually transient working memory. After a few sprints, these accumulate as dozens of `PLAN.md`/`SUMMARY.md` files that nobody owns and nobody reads.

## How to recognize AI junk

### 1. By filename

Common patterns generated by AI agents:

```
PLAN.md                          IMPLEMENTATION-PLAN.md
TODO.md                          IMPLEMENTATION-SUMMARY.md
NEXT-STEPS.md                    REFACTOR-PLAN.md
COMPLETED.md                     REFACTOR-NOTES.md
PROGRESS.md                      CHANGES.md  (when CHANGELOG.md already exists)
TASK-LIST.md                     SESSION-NOTES.md
CONTEXT.md                       DECISIONS.md (when docs/adr/ exists)
WORK-LOG.md                      DEBUG-NOTES.md
```

### 2. By content fingerprint

AI-generated docs often have telltale signatures:

- Trailing line: **"🤖 Generated with Claude Code"** / **"Generated by GitHub Copilot"** / **"Created by Codex"**
- Phrases like **"I have completed the following tasks"** or **"Here is a summary of the changes"**
- Bullet lists in past tense describing what was *just* done
- "Next steps:" sections that were never actioned
- Repeated boilerplate intros ("This document describes...")

### 3. By context

- File has 1 commit (created by the agent, never edited by a human)
- Last commit message: `"Add PLAN.md"` / `"Update progress"` / `"Add session summary"`
- Located at root of the repo (instead of properly organized under `docs/`)
- Date in the filename: `notes-2025-09-14.md`, `plan-q3-2025.md`

## Incorrect — what they look like

```markdown
❌ PLAN.md (committed by an agent, 8 months ago, never edited since)

# Implementation Plan

I'll be implementing the new user export feature in the following steps:

## Phase 1: Schema (Day 1)
- [x] Add export_jobs table
- [x] Add foreign key to users
- [x] Write migration

## Phase 2: Service layer (Day 2-3)
- [x] Create UserExportService
- [ ] Add streaming support

## Next steps
- Discuss approach with team
- Get sign-off before deployment

🤖 Generated with Claude Code
```

**Problems:**
- This was tracking work-in-progress; the work shipped 8 months ago
- The unchecked item is either obsolete (already done in a later PR) or forgotten
- File pollutes the repo's root and gets indexed by GitHub search
- Future agents read it and treat it as authoritative current state

## Correct — how to triage

For each candidate junk file, ask:

1. **Is the work in this file complete?** → Delete. Work history is in git/PRs.
2. **Is the work *abandoned*?** → Delete. (If you want to remember it, open an issue.)
3. **Is the work ongoing?** → Move it to the issue tracker, not a markdown file in the repo.
4. **Does it contain unique knowledge** (architecture decision, gotcha, runbook)? → Refactor into a proper doc:
   - Architecture decision → `docs/adr/NNNN-...md`
   - Gotcha / how-to → `docs/guides/...md`
   - Ops procedure → `docs/runbooks/...md`

**Never auto-delete.** Always surface for the user's approval first.

## Detection

```bash
# Filename patterns
find . -maxdepth 3 -type f -name '*.md' \
  -not -path './node_modules/*' -not -path './vendor/*' \
  | grep -Ei \
    '(PLAN|TODO|SUMMARY|PROGRESS|NEXT-STEPS|COMPLETED|TASK-LIST|SESSION-NOTES|WORK-LOG|REFACTOR-NOTES|DEBUG-NOTES|CONTEXT|CHANGES)\.md$'

# Content fingerprints — AI footer signatures
grep -rln -E '🤖 (Generated|Created) with|Co-Authored-By: Claude|Generated by (Copilot|Codex|Cursor)' \
  --include='*.md' .

# Markdown files committed by an agent and never edited by a human (single commit on the file).
# Useful for catching transient plan/summary files that landed and were forgotten.
find . -name '*.md' -not -path './node_modules/*' -not -path './vendor/*' | while read f; do
  COMMITS=$(git log --oneline -- "$f" | wc -l | tr -d ' ')
  [ "$COMMITS" = "1" ] && echo "SINGLE-COMMIT (likely auto-generated): $f"
done
```

## Prevention

- **Tell agents not to create plan files**: in your `CLAUDE.md` / `AGENTS.md` / `cursor.json`, instruct: "Do not create PLAN.md, TODO.md, or progress-tracking markdown files in the repo. Use the conversation context or the issue tracker."
- **CI gate**: block PRs that add `PLAN.md` / `TODO.md` / `SUMMARY.md` / etc. at root
- **Periodic audit**: every quarter, run the detection above

```yaml
# .github/workflows/no-agent-junk.yml
- name: Block AI-junk filenames
  run: |
    NEW=$(git diff --name-only --diff-filter=A origin/main...HEAD)
    BAD=$(echo "$NEW" | grep -Ei '^(PLAN|TODO|SUMMARY|PROGRESS|NEXT-STEPS)\.md$')
    test -z "$BAD" || { echo "Don't commit transient plan/summary files: $BAD"; exit 1; }
```

Reference: [Diátaxis — "what documentation is not"](https://diataxis.fr/) · [Internal: cleanup-orphans, cleanup-empty-stubs]

---


## Duplicate Documentation

**Impact: HIGH (Two copies of the same doc inevitably drift; readers can't tell which is current)**

Duplicate docs are the documentation equivalent of code duplication: every update has to happen in N places, and after the first inconsistency, no reader can tell which version is authoritative. Common causes are agents creating a "v2" of an existing doc, refactors that didn't delete the predecessor, and forks that drift.

## How duplicates appear

### 1. Near-identical content under different filenames

```
docs/architecture.md
docs/architecture-overview.md
docs/system-design.md
```

Three files, ~70% overlapping content, last updated by different authors at different times.

### 2. README sections duplicated in docs/

```
README.md                        # has "## Installation" with 30 lines
docs/guides/getting-started.md   # has its own install section, slightly different
docs/guides/setup.md             # also has install steps, outdated
```

### 3. "Updated" doc next to the old one

```
docs/deployment.md
docs/deployment-v2.md
docs/deployment-NEW.md
```

Always pick one; one of them is lying.

### 4. ADR vs design doc covering the same decision

```
docs/adr/0007-choose-mysql.md
docs/architecture/database-choice.md
```

The ADR is the canonical record; the design doc duplicates it. Either delete the design doc or replace it with a one-line pointer to the ADR.

## Incorrect

```
❌ Three near-duplicates of "how to deploy"
README.md → ## Deployment (15 lines)
docs/guides/deployment.md (80 lines, slightly different commands)
docs/runbooks/deploy-production.md (60 lines, mostly the same as guides/deployment.md but with prod-specific notes)
```

**Problems:**
- A new engineer asks "how do I deploy?" — gets three different answers
- A change to deploy command must update all three (and usually doesn't)
- Each copy ages at a different rate

## Correct

```
✅ One canonical source; others link to it

README.md
  ## Deployment
  See [docs/runbooks/deploy-production.md](docs/runbooks/deploy-production.md).

docs/guides/deployment.md     ← DELETED, content merged into runbook

docs/runbooks/deploy-production.md     ← the authoritative source
  # Deploy to production
  …full procedure…
```

**Benefits:**
- One place to change
- Readers reach the canonical version regardless of where they started
- Future "is this current?" question has one file to check

## Resolution strategies

When you find duplicates:

1. **Pick a winner** — the most current/most detailed/best-located version
2. **Merge unique content** from the others into the winner
3. **Delete the losers** — or replace them with one-line pointers if their location was useful (e.g., README → docs/runbooks)
4. **Add a CI check** to prevent re-introduction (markdownlint can flag similar headings across files)

## Detection

```bash
# Find near-duplicate markdown files (jscpd can do this)
npx jscpd --languages markdown --min-lines 20 docs/ README.md

# Or compare line-by-line for high similarity (bash; needs globstar for **)
shopt -s globstar nullglob
for a in docs/**/*.md; do
  for b in docs/**/*.md; do
    [[ "$a" < "$b" ]] || continue
    SIM=$(diff <(sort "$a") <(sort "$b") | wc -l)
    LINES=$(wc -l < "$a")
    if [ "$LINES" -gt 50 ] && [ "$SIM" -lt $((LINES / 4)) ]; then
      echo "SIMILAR: $a <-> $b (diff: $SIM lines)"
    fi
  done
done

# Files with the same H1 title
grep -rh '^# ' docs/ | sort | uniq -d
```

Reference: [Diátaxis — "one job per document"](https://diataxis.fr/) · [jscpd](https://github.com/kucherenko/jscpd)

---


## Orphaned Documentation

**Impact: MEDIUM (Docs nobody links to are docs nobody finds — they age into landmines)**

An orphan is a markdown file that no other file links to. It exists on disk, gets indexed by `git grep`, but nobody navigates to it via the normal docs flow. Orphans are usually either: drafts that were abandoned, docs whose linker was deleted, or junk that was never properly integrated.

## How to spot orphans

The simplest definition: **for every `.md` file in `docs/`, at least one *other* `.md` file should link to it**, directly or transitively from README.md.

Files that pass:
- `README.md` links to `docs/architecture/overview.md` ✓
- `docs/architecture/overview.md` links to `docs/architecture/data-model.md` ✓
- `docs/adr/0001-...md` is linked from `docs/adr/README.md` (or implicitly from numbering) ✓

Files that *fail*:
- `docs/old-onboarding-notes.md` — no inbound link
- `docs/api-thoughts.md` — was linked from a deleted README section
- `docs/guides/payments-deep-dive.md` — written 2 years ago, never linked

## Incorrect

```
❌ Orphan piles up while linked docs stay current
docs/
├── architecture/
│   └── overview.md             ← linked from README
├── guides/
│   ├── deployment.md           ← linked
│   ├── getting-started.md      ← linked
│   ├── old-onboarding.md       ← ORPHAN (no inbound links)
│   ├── api-thoughts.md         ← ORPHAN
│   └── payments-deep-dive.md   ← ORPHAN
└── archive/
    └── ...
```

**Problems:**
- Orphans show up in `grep` results and confuse readers who land on them
- Search engines (GitHub search, MkDocs site search) surface them
- Future "is this still relevant?" question has no obvious answer

## Correct — resolution options

For each orphan:

1. **Link it** — if the content is current and useful, add an inbound link from the closest hub (README, `docs/README.md`, the relevant guide). Then it's no longer an orphan.
2. **Archive it** — if it was once useful but no longer is, move to `docs/archive/<year>/`. The archive folder is *intentionally* unlinked from the main docs.
3. **Delete it** — if it's a draft, near-duplicate, or AI junk, delete. Git history preserves it.

**Never auto-delete.** Always surface for the user's approval.

## Detection

```bash
# Find all .md files in docs/, then check whether any other .md links to them.
# Uses two grep passes (one per pattern) since -F (fixed-string) doesn't support
# alternation; bare paths with `.` characters would mis-trigger with -E.
find docs/ -name '*.md' -type f | while read f; do
  base=$(basename "$f" .md)
  rel=${f#./}                       # path without leading ./
  REFS=$( { grep -rlF "$rel" --include='*.md' \
              --exclude-dir=node_modules --exclude-dir=vendor . ;
            grep -rlF "$base" --include='*.md' \
              --exclude-dir=node_modules --exclude-dir=vendor . ; } \
         | sort -u | grep -v "^$f$" )
  [ -z "$REFS" ] && echo "ORPHAN: $f"
done
```

Tools that do this better:

- **lychee** — checks for broken links; combined with a "list all referenced files" pass, can surface orphans
- **markdown-link-check** — per-file link checker
- **MkDocs / Docusaurus** — build-time warnings for unreferenced pages (with strict mode)

## Archived docs are *intentionally* orphans

`docs/archive/` is the one place where being unlinked is correct. The archive holds superseded content for historical reference; it should NOT be linked from current docs (otherwise readers might follow a stale link). Detection should exclude `docs/archive/`.

```bash
find docs/ -name '*.md' -not -path 'docs/archive/*' …
```

Reference: [lychee](https://github.com/lycheeverse/lychee) · [markdown-link-check](https://github.com/tcort/markdown-link-check)

---


## Empty Stubs and Placeholder Files

**Impact: MEDIUM (Stubs make the docs tree look complete when it isn't — false confidence)**

An empty stub is a markdown file that exists but contains nothing useful: just a heading, just "TBD", just a placeholder paragraph. Stubs are dangerous because they make the docs index look populated while delivering nothing — readers click expecting content and get an empty page, eroding trust in the entire docs.

## How to recognize stubs

### 1. Files with only a heading

```markdown
# Deployment Guide

```

(One line, then nothing. 100% useless.)

### 2. Files with "TBD" / "Coming soon" / placeholder content

```markdown
# API Reference

TBD — will document this later.
```

```markdown
# Architecture Overview

This document describes the architecture of the system.

(coming soon)
```

### 3. Files with only Lorem ipsum / template content

```markdown
# Title

Lorem ipsum dolor sit amet, consectetur adipiscing elit...
```

### 4. Files with the rule-template unchanged

(A bigger problem in skill repos like this one) — an `.md` file containing the literal `## Rule Title Here` template heading that was never edited.

### 5. Tiny files (< 30 lines, almost no information)

A 12-line `architecture.md` that just says "We use Laravel and React" is a stub even if it doesn't say "TBD".

## Incorrect

```markdown
❌ A stub that pretends to be documentation
docs/guides/deployment.md (the entire file):

# Deployment

TBD
```

**Problems:**
- A reader clicking this link gets nothing
- The file's existence implies the topic is documented when it isn't
- The "TBD" was written 18 months ago and forgotten

## Correct — three options

1. **Fill it in** — if you're going to write the doc, just write it (or at least cover the basics in 50+ lines). Don't create an empty file as a "reminder".

2. **Delete it** — if there's no plan to fill it in, delete. Reduce false expectations.

3. **Promote to an issue** — if you genuinely want to track "we need to write this someday", open a GitHub issue. The issue is for tracking; the markdown file is for content. Don't conflate them.

```markdown
✅ Replace with content OR delete
docs/guides/deployment.md (replacement):

# Deployment

Production deploys happen automatically when a tag is pushed to `main`:

1. Run tests locally: `php artisan test`
2. Bump the version in `composer.json`
3. Update CHANGELOG.md under `[Unreleased]`
4. Tag: `git tag v1.2.3 && git push --tags`
5. CI builds, tests, and deploys to production within 10 minutes
…etc, with real content.
```

## Detection

```bash
# Files under 30 lines (likely stubs)
find docs/ -name '*.md' -type f -exec wc -l {} \; | \
  awk '$1 < 30 { print }' | sort -n

# Files containing "TBD" / "Coming soon" / "Lorem"
grep -rlEi '\b(TBD|Coming soon|Lorem ipsum|placeholder|to be (done|written|filled))\b' \
  --include='*.md' docs/

# Files matching the rule-template heading (in this skills repo)
grep -rln '^## Rule Title Here$' --include='*.md' rules/

# Files with only a title and no content
for f in $(find docs/ -name '*.md'); do
  CONTENT_LINES=$(grep -cvE '^(#|$)' "$f")
  [ "$CONTENT_LINES" -lt 3 ] && echo "NEAR-EMPTY: $f"
done
```

## Prevention: don't create empty files

A common anti-pattern is creating the docs/ skeleton up-front: `touch docs/architecture.md docs/deployment.md docs/api.md`. This produces nothing but stubs.

Instead: create each doc file *when you have content for it*. The folder structure can be empty until then.

Reference: [Diátaxis — "documentation is not a TODO list"](https://diataxis.fr/)

---


## Freshness Dates on Architecture Docs

**Impact: MEDIUM (Without a freshness signal, every reader has to guess if the doc is still accurate)**

An architecture doc from 2022 might still be accurate or might be wildly out of date — without an explicit "Last verified" line, the reader can't tell. Adding a freshness date converts a doc from "trust at your own risk" to "verified accurate as of YYYY-MM-DD".

## What needs a freshness date

| Doc type | Freshness needed? |
|---|---|
| `docs/architecture/*` | **Yes** — these describe how the system *currently* works |
| `docs/guides/*` | **Yes** — setup steps change |
| `docs/runbooks/*` | **Yes** — procedures must be verified |
| `docs/api/*` | If hand-written; not if auto-generated from code |
| `docs/adr/*` | **No** — ADRs are dated by design (Date: field); they describe a past decision |
| `docs/archive/*` | **No** — archived = frozen in time |
| `README.md` | The "Installation" section — yes |

## Correct

```markdown
# Architecture Overview

_Last verified on a fresh checkout: 2026-04-12 by @asyraf_

## System layout

The application is a Laravel monolith with an Inertia.js + React frontend …
```

Or as a table at the top:

```markdown
| Field | Value |
|---|---|
| Last verified | 2026-04-12 |
| Verified by   | @asyraf |
| Owner         | @org/platform |
```

## Incorrect

```markdown
❌ No freshness signal anywhere
# Architecture Overview

The application uses MySQL for orders and Redis for sessions.
…
```

(Was that true in 2022? Still true today? You'd have to ask.)

## When to refresh

Set a cadence per doc-type:

- **Architecture docs** — quarterly (or when a major change ships)
- **Runbooks** — verify by *executing* them quarterly (a "walk-through" exercise)
- **Setup / install guides** — verify by running them on a clean checkout quarterly
- **API references** — when they're hand-written, every time a public endpoint changes

The freshness date is part of the doc's content; updating it is part of the verification work, not separate paperwork.

## Detection

```bash
# Find architecture / guide / runbook docs without a "Last verified" line
for f in docs/architecture/*.md docs/guides/*.md docs/runbooks/*.md; do
  grep -qE '_?Last verified' "$f" || echo "NO FRESHNESS DATE: $f"
done

# Find docs whose "Last verified" date is > 12 months ago.
# Compute threshold in bash (BSD awk on macOS doesn't have strftime/systime).
THRESHOLD=$(date -v-12m +%F 2>/dev/null || date -d '12 months ago' +%F)
grep -rEoH 'Last verified[: ]+[0-9]{4}-[0-9]{2}-[0-9]{2}' docs/ \
  | awk -F: -v t="$THRESHOLD" '$NF < t { print "STALE: " $0 }'
```

## CI: prompt-rather-than-block

Don't block PRs on freshness dates — that creates noise. Instead, prompt with a weekly digest:

```yaml
# .github/workflows/docs-freshness.yml — runs weekly
- name: List stale docs
  run: |
    THRESHOLD=$(date -d '12 months ago' +%F)
    grep -rEoH 'Last verified[: ]+[0-9]{4}-[0-9]{2}-[0-9]{2}' docs/ \
      | awk -F'verified[: ]+' -v t="$THRESHOLD" '$2 < t { print }' \
      | tee stale-docs.txt
    # Open a GitHub issue listing stale docs (or post to Slack)
```

Reference: [Diátaxis — keeping documentation maintained](https://diataxis.fr/) · Internal: `docs-outdated-architecture` rule in the [technical-debt](../../technical-debt) skill covers detection of stale content

---


## Archive Workflow for Superseded Docs

**Impact: MEDIUM (Keep history without polluting current docs; deletion is reversible via git, but archive is more discoverable)**

Some docs become wrong over time but contain context that's still useful: old architecture diagrams, prior deployment procedures, RFCs that were rejected. Don't delete them (you lose discoverability) and don't leave them in the current docs (you mislead readers). Move them to `docs/archive/<year>/` with a clear archive note.

## The archive folder

```
docs/archive/
├── 2024/
│   ├── q3-launch-plan.md
│   ├── old-architecture-overview.md      ← superseded by docs/architecture/overview.md
│   └── deprecated-deployment.md
└── 2025/
    ├── mysql-8-upgrade-plan.md           ← upgrade completed; plan kept for history
    └── rejected-graphql-rfc.md
```

Rules for the archive:

1. **Sub-foldered by year** — keeps it scannable as the archive grows
2. **Not linked from current docs** — being unlinked is the point (see `cleanup-orphans` for the exception)
3. **First line of archived doc states why** — see the template below

## Archive note (required at top of every archived doc)

```markdown
> **Archived 2025-11-03.** Superseded by [docs/architecture/overview.md](../../architecture/overview.md).
> Kept for historical context only. Do not follow procedures or rely on facts in this doc.

# Old Architecture Overview

(Original content follows unchanged.)
…
```

The archive note has three jobs:
- **Date** — when this was moved (not when it was written)
- **Superseded by** — link to the current canonical version (if one exists)
- **Warning** — explicit "don't use this" so misreaders self-correct

## Incorrect

```
❌ Renaming instead of archiving
docs/architecture-OLD.md            (clutters current docs/)
docs/architecture-archive.md        (still in current docs/)
docs/old-deployment-DEPRECATED.md
```

(These are just renamed-in-place; readers and tools still find them mixed with current docs.)

```
❌ Outright deletion of unique context
$ git rm docs/mysql-8-upgrade-plan.md   # plan that documented WHY the upgrade was painful
```

(Git history preserves the file, but nobody will think to look for it via `git log`.)

## Correct

```bash
# Move to archive with a one-line PR message
mkdir -p docs/archive/2025
git mv docs/architecture-overview.md docs/archive/2025/old-architecture-overview.md
$EDITOR docs/archive/2025/old-architecture-overview.md  # add the archive note at top
git commit -m "docs: archive old architecture overview (superseded by new overview)"
```

The new canonical doc (`docs/architecture/overview.md`) was either created in a separate PR or in the same PR as the archive move.

## When to archive vs delete

| Situation | Action |
|---|---|
| Old doc replaced by a new one, but the old contains historical decisions / context | Archive |
| Migration plan that succeeded — the plan itself documents trade-offs | Archive |
| Rejected RFC — the rejection rationale is useful history | Archive |
| AI-generated plan file that was never relevant to begin with | Delete (see `cleanup-ai-junk`) |
| Empty stub | Delete (see `cleanup-empty-stubs`) |
| Near-duplicate of another doc with no unique content | Delete (see `cleanup-duplicates`) |
| Personal notes / scratchpad | Delete |

**Tip:** when in doubt, archive rather than delete. Deletion is final-feeling for readers (git history exists but is invisible); archive keeps the file visible to anyone browsing.

## Detection — what's in the archive that shouldn't be

```bash
# Archived docs missing the archive note (bash; needs globstar for **)
shopt -s globstar nullglob
for f in docs/archive/**/*.md; do
  head -3 "$f" | grep -q 'Archived' || echo "MISSING ARCHIVE NOTE: $f"
done

# Archived docs linked from CURRENT docs (mistake — current docs should link to current docs).
# Filter on the source-file path, not on the matched line.
find docs/ -name '*.md' -not -path 'docs/archive/*' -print0 | \
  xargs -0 grep -lE 'docs/archive/'
```

Reference: Diátaxis · [Internal: cleanup-orphans (archive is the intentional-orphan exception)]

---


## ADR Process — Proposed → Accepted → Superseded

**Impact: HIGH (Architecture decisions need a paper trail; without it, the same debates repeat every year)**

ADRs (Architecture Decision Records) document **why** a non-trivial technical decision was made. The format is append-only with explicit status transitions, so future readers can trace the reasoning *and* see whether each decision is still in force.

## What deserves an ADR

Write an ADR when:

- **The decision is hard to reverse** — choice of database, framework, monorepo vs polyrepo
- **It will be questioned later** — "why did we pick MySQL over Postgres?"
- **There are real trade-offs** — multiple defensible options
- **It affects code structure** for years to come

Don't write an ADR for:

- Small implementation choices (which loop pattern, which formatting)
- Decisions documented better elsewhere (security policies → `SECURITY.md`)
- Tactical config (which lint rule severity)

A good rule of thumb: if you can explain the choice in a commit message, you don't need an ADR.

## Status lifecycle

```
proposed  →  accepted  →  superseded
                  ↘  rejected
                  ↘  deprecated
```

| Status | Meaning |
|---|---|
| **Proposed** | Drafted, under team review |
| **Accepted** | Active — this is what we do |
| **Rejected** | Considered and dismissed (the reasoning is still useful) |
| **Deprecated** | No longer the recommended approach, but legacy code may still follow it |
| **Superseded by NNNN** | Replaced by a later ADR — link to it |

Status is part of the **content**, not the filename. Don't rename `0007-...` to `0007-DEPRECATED-...`. Update the `Status:` line.

## ADR template (Michael Nygard's, lightly extended)

```markdown
# 0007. Rate-limit the public API

| Field   | Value |
|---------|-------|
| Date    | 2026-05-16 |
| Status  | Accepted |
| Owner   | @org/platform |

## Context

What forces are at play? What problem are we solving? What constraints exist?

(2–4 paragraphs)

## Decision

What did we decide to do? Be specific and concrete.

(1–2 paragraphs)

## Consequences

What becomes easier as a result? What becomes harder? Any follow-on work?

- Easier: …
- Harder: …
- Follow-up: …

## Alternatives considered

- **Option A** — rejected because …
- **Option B** — rejected because …
- **Status quo (do nothing)** — rejected because …

## References

- Linked issue/PR: #1234
- Related ADRs: 0003, 0009
```

## When superseding an ADR

1. **Write a new ADR** (e.g., `0019`) that describes the new decision
2. **Update the old ADR's status** to `Superseded by ADR-0019` and link to it
3. **Do NOT delete or rewrite the old ADR** — it remains as historical record

```markdown
# 0007. Rate-limit the public API

| Field   | Value |
|---------|-------|
| Date    | 2026-05-16 |
| Status  | Superseded by [ADR-0019](0019-tiered-rate-limit-with-redis.md) |
| Owner   | @org/platform |

(Original content unchanged below.)
…
```

## Incorrect

```
❌ Decisions live only in Slack and PR descriptions

(Team picks Redis over Memcached in a Slack thread; 6 months later, three new
engineers ask why, and one of them re-litigates the decision.)
```

```
❌ Status managed via filename renames

docs/adr/0007-rate-limit-DEPRECATED.md      # makes the file harder to reference
docs/adr/0007-rate-limit-SUPERSEDED.md      # breaks links from older code/PRs
```

```
❌ Deleting superseded ADRs

git rm docs/adr/0007-...                    # erases the "why we used to do it differently"
```

## Detection

```bash
# ADRs without an explicit status line (covers both prose `Status: ...` and table `| Status | ... |`)
for f in docs/adr/[0-9]*.md; do
  grep -qiE '^(Status[:* ]|\| *Status)' "$f" || echo "NO STATUS: $f"
done

# ADRs claiming Superseded without a link (handles both prose and table-format)
grep -lEi '(^|\| *)Status[:|* ]+.*Superseded' docs/adr/*.md | while read f; do
  grep -Ei 'Superseded by[: ]+\[?[Aa][Dd][Rr]-?[0-9]+' "$f" >/dev/null \
    || echo "MISSING SUPERSEDED LINK: $f"
done
```

Reference: [adr.github.io](https://adr.github.io/) · [Michael Nygard's original post](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) · [adr-tools CLI](https://github.com/npryce/adr-tools)

---


## CHANGELOG-in-the-PR Discipline

**Impact: HIGH (CHANGELOG entries written after the fact are always wrong; do it with the change)**

A CHANGELOG only stays accurate if entries are written **in the same PR as the change**. The instant you defer to "we'll batch this before release", entries get missed, misattributed, or rewritten from memory inaccurately. The discipline is: every non-trivial PR adds a line under `[Unreleased]`.

## The rule

For every PR that is **not** purely:
- documentation-only
- test-only
- internal refactor (no behavior change visible to users)
- trivial dependency bump (no breaking changes)

…the PR must include a CHANGELOG entry under `[Unreleased]`.

## Incorrect

```
❌ PR description: "We'll add the CHANGELOG entry before the release"

(Three months later: 47 PRs have shipped, half of them touched user-visible
behavior, the release manager is reconstructing CHANGELOG entries from
PR titles, and three changes are missing entirely.)
```

```
❌ Generic CHANGELOG entries

## [Unreleased]
### Changed
- Various improvements and bug fixes
- Updated dependencies
- Refactored internals
```

(Tells the reader nothing actionable. May as well be empty.)

## Correct — entry written in the PR that ships the change

```markdown
## [Unreleased]

### Added
- Bulk user export endpoint (#412): `GET /api/users/export?format=csv` streams
  up to 1M rows without buffering.

### Changed
- Password hashing default switched from bcrypt to argon2id for new users.
  Existing bcrypt hashes are still accepted and re-hashed transparently on
  next successful login. (#421)

### Fixed
- Cursor pagination dropping the last row of each page (#387).
```

Each entry tells the user **what changed** and **why they might care**.

## CI enforcement

Block PRs that don't add a CHANGELOG entry, with a skip-label for trivial PRs:

```yaml
# .github/workflows/changelog.yml
name: CHANGELOG check
on: pull_request

jobs:
  changelog:
    runs-on: ubuntu-latest
    if: "!contains(github.event.pull_request.labels.*.name, 'skip-changelog')"
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - name: Require CHANGELOG entry
        run: |
          # Skip if PR only touches docs/tests/CI
          CHANGED=$(git diff --name-only origin/${{ github.base_ref }}...HEAD)
          IGNORE='^(\.github/|docs/|tests?/|README\.md|\.gitignore|CHANGELOG\.md)'
          NON_TRIVIAL=$(echo "$CHANGED" | grep -vE "$IGNORE" || true)
          if [ -n "$NON_TRIVIAL" ]; then
            CHANGELOG_DIFF=$(git diff origin/${{ github.base_ref }}...HEAD -- CHANGELOG.md)
            echo "$CHANGELOG_DIFF" | grep -qE '^\+[^+]' \
              || { echo "::error::Add a CHANGELOG.md entry under [Unreleased] (or label 'skip-changelog')"; exit 1; }
          fi
```

The `skip-changelog` label is for genuine exceptions (docs-only, test-only). Don't abuse it.

## On release: cut [Unreleased]

When releasing a new version:

1. Rename `[Unreleased]` to `[1.4.0] - 2026-05-16`
2. Add a fresh empty `## [Unreleased]` heading on top
3. Update the compare-link footnotes:

```markdown
[Unreleased]: https://github.com/org/repo/compare/v1.4.0...HEAD
[1.4.0]:      https://github.com/org/repo/compare/v1.3.0...v1.4.0
```

4. Commit with `chore(release): v1.4.0`
5. Tag: `git tag v1.4.0`

## Entry-writing tips

- **Subject of the sentence is what changed**, not "we": "Added X" not "We added X"
- **One PR = one bullet** in most cases; only split if the PR ships two distinct changes
- **Link the PR or issue** in parentheses at the end
- **Don't include refactors that aren't user-visible** — they belong in commit messages, not CHANGELOG

Reference: [Keep a Changelog](https://keepachangelog.com/) · [Conventional Commits → CHANGELOG generation](https://github.com/conventional-changelog/conventional-changelog) (if you prefer auto-generated)

---

