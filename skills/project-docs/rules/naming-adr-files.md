---
title: ADR File Naming — Numbered Prefix
impact: HIGH
impactDescription: "ADRs are append-only history; numbering makes order, references, and freshness obvious"
tags: naming, adr, architecture-decision-record
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
