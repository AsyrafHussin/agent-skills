---
title: Archive Workflow for Superseded Docs
impact: MEDIUM
impactDescription: "Keep history without polluting current docs; deletion is reversible via git, but archive is more discoverable"
tags: lifecycle, archive, history
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
