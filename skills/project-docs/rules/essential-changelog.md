---
title: CHANGELOG — Keep-a-Changelog Format
impact: HIGH
impactDescription: "Release history readers can actually parse; the alternative is reading every commit"
tags: essential, changelog, releases
---

## CHANGELOG — Keep-a-Changelog Format

**Impact: HIGH (Release history readers can actually parse; the alternative is reading every commit)**

Every project past its first release needs a CHANGELOG that humans can read in 30 seconds. "What changed in v3.2?" should not require `git log v3.1..v3.2`. The [Keep a Changelog](https://keepachangelog.com/) format is the de-facto standard: one section per version, grouped by change type, newest first.

## Required structure

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- Bullet for each new feature in flight

## [1.3.0] - 2026-05-10

### Added
- Bulk user export endpoint (#412)
- MySQL 8.4 support; bumped CI to test against it

### Changed
- Password hashing default switched to argon2id (existing bcrypt hashes still validated and re-hashed on login)

### Fixed
- Stripe webhook idempotency key handling (#398, #401)

### Security
- Bumped guzzlehttp/guzzle to 7.9.2 (CVE-2024-XXXX)

## [1.2.1] - 2026-04-22

### Fixed
- Cursor pagination skipping the last row of each page (#387)

## [1.2.0] - 2026-04-01

### Added
- Initial release of public REST API

[Unreleased]: https://github.com/org/repo/compare/v1.3.0...HEAD
[1.3.0]: https://github.com/org/repo/compare/v1.2.1...v1.3.0
[1.2.1]: https://github.com/org/repo/compare/v1.2.0...v1.2.1
[1.2.0]: https://github.com/org/repo/releases/tag/v1.2.0
```

## Change-type buckets (use these)

| Bucket | When to use |
|---|---|
| **Added** | New features or capabilities |
| **Changed** | Changes to existing functionality |
| **Deprecated** | Soon-to-be-removed features (announce here, then remove next major) |
| **Removed** | Removed features |
| **Fixed** | Bug fixes |
| **Security** | CVE patches, vulnerability fixes |

Drop unused buckets from a given release (don't include empty `### Added` if there were no additions).

## Incorrect

```markdown
❌ Free-form CHANGELOG that nobody can scan

# Changes

v1.3 — May 2026 — lots of stuff, see commits for details
v1.2 — April 2026 — bugfixes and improvements
v1.1 — March 2026 — added cool feature
v1.0 — Feb 2026 — initial release
```

**Problems:**
- "Lots of stuff" tells the reader nothing
- No buckets, so security fixes mix with cosmetic changes
- No dates in ISO format (international ambiguity)
- No links to commits, PRs, or version tags

## CHANGELOG-in-the-PR discipline

A CHANGELOG entry should land in **the same PR** as the change it describes — not after the fact, not "later in a batch". This is the only way to keep it accurate.

Enforce with CI:

```yaml
# .github/workflows/changelog.yml
- name: CHANGELOG entry required for non-trivial changes
  run: |
    if git diff --name-only origin/main...HEAD | grep -qvE '^(\.github/|docs/|tests?/|README\.md|\.gitignore)$'; then
      git diff origin/main...HEAD -- CHANGELOG.md | grep -q '^+' \
        || { echo "Add a CHANGELOG.md entry"; exit 1; }
    fi
```

Skip for docs-only, test-only, and trivial PRs (use a label or PR title prefix).

## When to start a CHANGELOG

- **Library / package** — from the very first published version
- **Application** — from the first deployment to production OR the first release tagged with `v`
- **Internal tool** — when more than one team consumes it

For pre-1.0 projects in flux, [Keep a Changelog](https://keepachangelog.com/en/1.1.0/#how) explicitly endorses starting with `0.x.y` and using `[Unreleased]` heavily.

Reference: [Keep a Changelog](https://keepachangelog.com/) · [Semantic Versioning](https://semver.org/)
