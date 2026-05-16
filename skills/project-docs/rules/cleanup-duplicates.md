---
title: Duplicate Documentation
impact: HIGH
impactDescription: "Two copies of the same doc inevitably drift; readers can't tell which is current"
tags: cleanup, duplicates, consolidation
---

## Duplicate Documentation

**Impact: HIGH (Two copies of the same doc inevitably drift; readers can't tell which is current)**

Duplicate docs are the documentation equivalent of code duplication: every update has to happen in N places, and after the first inconsistency, no reader can tell which version is authoritative. Common causes are agents creating a "v2" of an existing doc, refactors that didn't delete the predecessor, and forks that drift.

## How duplicates appear

### 1. Near-identical content under different filenames

```
docs/architecture.md
docs/architecture-overview.md
docs/system-design.md
```

Three files, ~70% overlapping content, last updated by different authors at different times.

### 2. README sections duplicated in docs/

```
README.md                        # has "## Installation" with 30 lines
docs/guides/getting-started.md   # has its own install section, slightly different
docs/guides/setup.md             # also has install steps, outdated
```

### 3. "Updated" doc next to the old one

```
docs/deployment.md
docs/deployment-v2.md
docs/deployment-NEW.md
```

Always pick one; one of them is lying.

### 4. ADR vs design doc covering the same decision

```
docs/adr/0007-choose-mysql.md
docs/architecture/database-choice.md
```

The ADR is the canonical record; the design doc duplicates it. Either delete the design doc or replace it with a one-line pointer to the ADR.

## Incorrect

```
❌ Three near-duplicates of "how to deploy"
README.md → ## Deployment (15 lines)
docs/guides/deployment.md (80 lines, slightly different commands)
docs/runbooks/deploy-production.md (60 lines, mostly the same as guides/deployment.md but with prod-specific notes)
```

**Problems:**
- A new engineer asks "how do I deploy?" — gets three different answers
- A change to deploy command must update all three (and usually doesn't)
- Each copy ages at a different rate

## Correct

```
✅ One canonical source; others link to it

README.md
  ## Deployment
  See [docs/runbooks/deploy-production.md](docs/runbooks/deploy-production.md).

docs/guides/deployment.md     ← DELETED, content merged into runbook

docs/runbooks/deploy-production.md     ← the authoritative source
  # Deploy to production
  …full procedure…
```

**Benefits:**
- One place to change
- Readers reach the canonical version regardless of where they started
- Future "is this current?" question has one file to check

## Resolution strategies

When you find duplicates:

1. **Pick a winner** — the most current/most detailed/best-located version
2. **Merge unique content** from the others into the winner
3. **Delete the losers** — or replace them with one-line pointers if their location was useful (e.g., README → docs/runbooks)
4. **Add a CI check** to prevent re-introduction (markdownlint can flag similar headings across files)

## Detection

```bash
# Find near-duplicate markdown files (jscpd can do this)
npx jscpd --languages markdown --min-lines 20 docs/ README.md

# Or compare line-by-line for high similarity (bash; needs globstar for **)
shopt -s globstar nullglob
for a in docs/**/*.md; do
  for b in docs/**/*.md; do
    [[ "$a" < "$b" ]] || continue
    SIM=$(diff <(sort "$a") <(sort "$b") | wc -l)
    LINES=$(wc -l < "$a")
    if [ "$LINES" -gt 50 ] && [ "$SIM" -lt $((LINES / 4)) ]; then
      echo "SIMILAR: $a <-> $b (diff: $SIM lines)"
    fi
  done
done

# Files with the same H1 title
grep -rh '^# ' docs/ | sort | uniq -d
```

Reference: [Diátaxis — "one job per document"](https://diataxis.fr/) · [jscpd](https://github.com/kucherenko/jscpd)
