---
title: Naming Anti-Patterns to Reject
impact: HIGH
impactDescription: "Junk names accumulate fast — once you accept one, the floodgates open"
tags: naming, anti-patterns, cleanup
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
