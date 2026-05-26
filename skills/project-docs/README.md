# Project Documentation

End-to-end documentation lifecycle for **PHP/Laravel and Node/TypeScript/React** projects. Covers folder structure, naming conventions, essential files, freshness, and cleanup of AI-generated junk. Supports bootstrap, audit, and reference modes.

**Version:** 1.0.0

## Overview

- Bootstrap a new project's docs (what's needed + scaffolded templates)
- Audit existing markdown files: keep / update / archive / delete / move
- Detect AI-generated plan/summary junk (PLAN.md, IMPLEMENTATION-SUMMARY.md, etc.)
- Enforce folder structure (`docs/architecture/`, `docs/adr/`, `docs/guides/`, `docs/runbooks/`, `docs/archive/`)
- Enforce naming (`UPPERCASE.md` for root files, `kebab-case.md` for docs, numbered ADRs)
- 25 rules across 6 categories

## Categories

### 1. Structure (CRITICAL)
Where docs live: root vs `docs/`; recommended sub-folder layout.

### 2. Naming (CRITICAL)
`UPPERCASE.md` for root, `kebab-case.md` for `docs/*`, numbered ADRs, no dates / no draft markers.

### 3. Essential Files (HIGH)
The minimum every project needs: README, CHANGELOG, LICENSE — plus CONTRIBUTING and SECURITY for projects that warrant them.

### 4. Quality (HIGH)
Conciseness over bloat, AI-slop fingerprints in prose, heading hierarchy, copy-pasteable code blocks, descriptive non-broken links.

### 5. Cleanup (HIGH)
Detect AI junk files, duplicates, orphans, and empty stubs; surface for triage without auto-deleting.

### 6. Lifecycle (MEDIUM)
Freshness dates, archive workflow, ADR process, CHANGELOG-on-PR discipline.

## Usage

```
Set up docs for this Laravel project
Audit the markdown files in this repo
Clean up the docs folder
What docs does this project need?
Where should this architecture diagram go?
How should I name this ADR?
Find stale documentation
```

## References

- [Keep a Changelog](https://keepachangelog.com/)
- [Architecture Decision Records (ADR)](https://adr.github.io/)
- [Diátaxis — documentation framework](https://diataxis.fr/)
- [Choose a License](https://choosealicense.com/)
- [Markdownlint](https://github.com/DavidAnson/markdownlint)
