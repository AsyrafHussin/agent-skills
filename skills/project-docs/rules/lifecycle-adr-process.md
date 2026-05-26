---
title: ADR Process — Proposed → Accepted → Superseded
impact: HIGH
impactDescription: "Architecture decisions need a paper trail; without it, the same debates repeat every year"
tags: lifecycle, adr, architecture-decision-record
---

## ADR Process — Proposed → Accepted → Superseded

**Impact: HIGH (Architecture decisions need a paper trail; without it, the same debates repeat every year)**

ADRs (Architecture Decision Records) document **why** a non-trivial technical decision was made. The format is append-only with explicit status transitions, so future readers can trace the reasoning *and* see whether each decision is still in force.

## What deserves an ADR

Write an ADR when:

- **The decision is hard to reverse** — choice of database, framework, monorepo vs polyrepo
- **It will be questioned later** — "why did we pick MySQL over Postgres?"
- **There are real trade-offs** — multiple defensible options
- **It affects code structure** for years to come

Don't write an ADR for:

- Small implementation choices (which loop pattern, which formatting)
- Decisions documented better elsewhere (security policies → `SECURITY.md`)
- Tactical config (which lint rule severity)

A good rule of thumb: if you can explain the choice in a commit message, you don't need an ADR.

## Status lifecycle

```
proposed  →  accepted  →  superseded
                  ↘  rejected
                  ↘  deprecated
```

| Status | Meaning |
|---|---|
| **Proposed** | Drafted, under team review |
| **Accepted** | Active — this is what we do |
| **Rejected** | Considered and dismissed (the reasoning is still useful) |
| **Deprecated** | No longer the recommended approach, but legacy code may still follow it |
| **Superseded by NNNN** | Replaced by a later ADR — link to it |

Status is part of the **content**, not the filename. Don't rename `0007-...` to `0007-DEPRECATED-...`. Update the `Status:` line.

## ADR template (Michael Nygard's, lightly extended)

```markdown
# 0007. Rate-limit the public API

| Field   | Value |
|---------|-------|
| Date    | 2026-05-16 |
| Status  | Accepted |
| Owner   | @org/platform |

## Context

What forces are at play? What problem are we solving? What constraints exist?

(2–4 paragraphs)

## Decision

What did we decide to do? Be specific and concrete.

(1–2 paragraphs)

## Consequences

What becomes easier as a result? What becomes harder? Any follow-on work?

- Easier: …
- Harder: …
- Follow-up: …

## Alternatives considered

- **Option A** — rejected because …
- **Option B** — rejected because …
- **Status quo (do nothing)** — rejected because …

## References

- Linked issue/PR: #1234
- Related ADRs: 0003, 0009
```

## When superseding an ADR

1. **Write a new ADR** (e.g., `0019`) that describes the new decision
2. **Update the old ADR's status** to `Superseded by ADR-0019` and link to it
3. **Do NOT delete or rewrite the old ADR** — it remains as historical record

```markdown
# 0007. Rate-limit the public API

| Field   | Value |
|---------|-------|
| Date    | 2026-05-16 |
| Status  | Superseded by [ADR-0019](0019-tiered-rate-limit-with-redis.md) |
| Owner   | @org/platform |

(Original content unchanged below.)
…
```

## Incorrect

```
❌ Decisions live only in Slack and PR descriptions

(Team picks Redis over Memcached in a Slack thread; 6 months later, three new
engineers ask why, and one of them re-litigates the decision.)
```

```
❌ Status managed via filename renames

docs/adr/0007-rate-limit-DEPRECATED.md      # makes the file harder to reference
docs/adr/0007-rate-limit-SUPERSEDED.md      # breaks links from older code/PRs
```

```
❌ Deleting superseded ADRs

git rm docs/adr/0007-...                    # erases the "why we used to do it differently"
```

## Detection

```bash
# ADRs without an explicit status line (covers both prose `Status: ...` and table `| Status | ... |`)
for f in docs/adr/[0-9]*.md; do
  grep -qiE '^(Status[:* ]|\| *Status)' "$f" || echo "NO STATUS: $f"
done

# ADRs claiming Superseded without a link (handles both prose and table-format)
grep -lEi '(^|\| *)Status[:|* ]+.*Superseded' docs/adr/*.md | while read f; do
  grep -Ei 'Superseded by[: ]+\[?[Aa][Dd][Rr]-?[0-9]+' "$f" >/dev/null \
    || echo "MISSING SUPERSEDED LINK: $f"
done
```

Reference: [adr.github.io](https://adr.github.io/) · [Michael Nygard's original post](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) · [adr-tools CLI](https://github.com/npryce/adr-tools)
