---
title: docs/ Sub-folder Layout
impact: HIGH
impactDescription: "Different doc kinds have different audiences and lifecycles — separate them"
tags: structure, docs-folder, organization, diataxis
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
