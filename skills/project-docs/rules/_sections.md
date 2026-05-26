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
