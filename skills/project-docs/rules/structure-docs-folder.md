---
title: docs/ as the Single Documentation Home
impact: CRITICAL
impactDescription: "One predictable home for everything beyond root files — no more 'where did we put that diagram?'"
tags: structure, docs-folder, organization
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
