---
title: Orphaned Documentation
impact: MEDIUM
impactDescription: "Docs nobody links to are docs nobody finds — they age into landmines"
tags: cleanup, orphans, link-graph
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
