# Code Slop Detection

Taste-level review of code for AI-generated patterns ("slop") in **PHP/Laravel and TypeScript/React** projects. Catches code that passes every metric but reads like a tutorial blog post — not what a human teammate would write.

**Version:** 1.0.0

## Overview

- Audits AI-assisted PRs and codebases for AI-fingerprint patterns
- Verdict bands: CLEAN / SUSPICIOUS / INFLATED / CRITICAL
- Stack: PHP / Laravel + Node / TypeScript / React
- 24 rules across 6 categories
- Complements `technical-debt` (quantitative metrics) with qualitative taste

## What it catches that linters don't

| Slop pattern | Why linters miss it |
|---|---|
| Narration comments | Looks like a "valid comment" to any linter |
| Generic names (`data`, `result`) | Passes naming conventions |
| Premature interfaces / single-method classes | Valid code structure |
| Generic `catch (e) { console.error(...) }` | Try/catch is a valid pattern |
| Mock-everything tests | Tests pass and run; coverage looks fine |
| Missing `// HACK:` scars | No tool checks for *absence* of human imperfection |

## Categories

### 1. Comments (CRITICAL)
Narration, empty docblocks, placeholder TODOs, closing-brace labels.

### 2. Naming (CRITICAL)
Generic placeholders, over-descriptive run-ons, suffix abuse, type-in-name.

### 3. Over-engineering (HIGH)
Premature interfaces, single-method classes, useless wrappers, dependency creep.

### 4. Defensive overdose (HIGH)
Generic catch blocks, impossible null checks, missing real defenses.

### 5. Test slop (HIGH)
Mock-everything, "doesn't throw" assertions, mirror-implementation, snapshot abuse.

### 6. Style fingerprints (MEDIUM)
Hyper-consistent formatting, `as any` escapes, no `// HACK:` scars, debug artifacts, trivial boilerplate.

## Usage

```
Review this PR for AI slop
Audit src/ for AI-generated code patterns
Does this code look human-written?
Find the slop in app/Services/
Check this file against the slop checklist
```

## Differentiator vs `technical-debt`

| Skill | Lens |
|---|---|
| `technical-debt` | Quantitative — complexity, duplication, CVEs, missing indexes |
| `code-slop` | Qualitative — does this code feel AI-written? |

Some overlap exists on dead code and generic catch blocks, but framing and remediation differ.

## References

- GitClear 2025 Trends Report
- [arXiv 2510.03029 — LLM-Generated Code Smells](https://arxiv.org/abs/2510.03029)
- [hardikpandya/stop-slop](https://github.com/hardikpandya/stop-slop) — prose-slop sister project
- [flamehaven01/AI-SLOP-Detector](https://github.com/flamehaven01/AI-SLOP-Detector) — AST scanner
