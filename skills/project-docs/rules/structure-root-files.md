---
title: Root-Level Documentation Files
impact: CRITICAL
impactDescription: "Discoverability — first impression for every new reader, every tool, every hosting platform"
tags: structure, root, readme, changelog
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
