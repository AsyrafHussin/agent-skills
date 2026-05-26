---
title: CHANGELOG-in-the-PR Discipline
impact: HIGH
impactDescription: "CHANGELOG entries written after the fact are always wrong; do it with the change"
tags: lifecycle, changelog, process, ci
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
