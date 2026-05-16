---
title: Hyper-Consistent Formatting
impact: MEDIUM
impactDescription: "A codebase where every file looks linter-perfect is suspiciously not-human"
tags: style, formatting, ai-fingerprint
---

## Hyper-Consistent Formatting

**Impact: MEDIUM (A codebase where every file looks linter-perfect is suspiciously not-human)**

Real codebases have geology — older files use older conventions, busy modules have rushed sections, 2am hotfix code has different spacing than carefully-reviewed weeks. AI-generated code is **uniformly polished**: every function the same length, every blank line in the same place, every import alphabetized, no styling drift anywhere. This isn't a hallmark of quality — it's a hallmark of generation.

The fingerprint is **uniformity at scale**. One file looking pristine is fine. A 2000-line PR where every file is identically pristine, across files that should have different velocity histories, is a strong AI signal.

## What it looks like

The PR diff shows:

- Every PHP file has exactly 4-space indent, exactly one blank line between methods, exactly two blank lines between class members, every parameter aligned identically
- Every TS file has the same import-grouping pattern, named imports always alphabetised, every arrow function body wrapped identically
- No `// HACK:` / `// XXX:` / late-night comments anywhere
- No commented-out code (clean — but also no signs of someone exploring)
- Identical commit-message structure across 50 commits
- All files use the same paradigm (e.g., every function is an arrow function; every class uses constructor property promotion) even in places where the existing codebase mixed styles

## Why this matters

Code geology is a tool for the next developer:
- **Different ages tell different stories** — "this section is from 2022, before we switched to..."
- **`// HACK:` markers signal known fragility** that hasn't been worth refactoring
- **Commented-out code reveals exploration** — "we tried X, didn't work, kept the option in case"
- **Velocity differences** show which paths were rushed and might harbour bugs

A repo with no geology forces every reader to start fresh — no inherited context, no "the team didn't bother fixing this, it works fine" signal. AI-generated codebases are flat: every line treated as equally important.

## The "all green field" tell

Hyper-consistency is most suspicious in **brownfield additions** — i.e., when AI adds code to a codebase that has its own conventions. Real engineers either:

- **Match the existing style** (consistent with the surrounding 50 files)
- **Refactor the surrounding files** alongside their change (drift announces itself in the diff)
- **Introduce a deliberate change** with an explanation ("new files use the new pattern; old files updated as touched")

AI typically does the first poorly — it picks "the most recent convention" or "the most popular convention from training data" rather than matching the actual codebase. Result: the new files are linter-perfect by some standard, but inconsistent with the surrounding codebase.

## What "human formatting" looks like

You won't catch this with a linter. It's a feel:

- Some files have slightly off blank-line spacing where someone hit enter twice
- Imports sometimes group "stuff I added recently" at the top
- One file has 6-space indent inside a heredoc because the dev didn't fight prettier on it
- A function has an extra blank line before a tricky if-branch where the author paused to think
- A 2-line comment is in mid-sentence-case because someone typed it angry

These tiny artifacts are how humans write code. Their absence — especially across many files at once — is the slop.

## Detection

There's no automatic detector. The signal is human-judgment:

1. **Run the diff through `git diff --stat`** — many files changed at once is normal; many files all with the same line-count profile is suspicious
2. **`git log --author` distribution** — if the diff is from one author but covers 30 files in one commit, raise an eyebrow
3. **Compare new files to surrounding files** — does the indentation, import order, brace style match the existing convention?
4. **Search for `// HACK:` / `// XXX:` markers** in the new files — their *absence* is the signal:
   ```bash
   git diff origin/main...HEAD -- '*.ts' '*.php' | grep -E '// HACK:|// XXX:|// FIXME' || echo "NO HACK/XXX MARKERS"
   ```
5. **Mixed-paradigm check** — in a TS PR, count `function` declarations vs arrow functions; in a PHP PR, count `final class` vs `class`. A sudden swing is a tell.

## What to do

If a PR looks hyper-consistent:

- Ask the author: "Was this AI-assisted? Walk me through the section in `OrderService` line 80." Their ability to explain the *intent*, not just the code, separates "AI-written, human-reviewed" from "AI-written, accepted-as-is".
- Sample a few functions and ask: "What's the trade-off you chose here?" If they shrug, the code wasn't really written by them.

The goal isn't to ban AI assistance — it's to make sure the author owns the code they ship.

Reference: Internal: [`style-no-hack-scars`](style-no-hack-scars.md)
