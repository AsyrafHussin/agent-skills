---
name: code-slop
description: Detect AI-generated code patterns ("slop") in PHP/Laravel and TypeScript/React source — comment narration, generic naming, premature interfaces, defensive overdose, mock-everything tests, and the absence of human "scars". Use when reviewing AI-assisted PRs, auditing code for taste/quality (not metrics — that's technical-debt), or hardening a code-review checklist. Triggers on "review for AI slop", "find AI patterns", "check code feels human", "audit code-quality taste".
license: MIT
metadata:
  author: agent-skills
  version: "1.0.0"
---

# Code Slop Detection

Taste-level review of code for AI-generated patterns. Contains 24 rules across 6 categories covering comments, naming, over-engineering, defensive overdose, test slop, and style fingerprints. Where [`technical-debt`](../technical-debt) measures *quantitative* code debt (complexity, duplication, CVEs), this skill measures the *qualitative* failure mode: code that passes every metric but reads like a tutorial blog post, not like a human wrote it.

## Metadata

- **Version:** 1.0.0
- **Scope:** PHP / Laravel + TypeScript / React (Node)
- **Rule Count:** 24 rules across 6 categories
- **License:** MIT

## Why this skill exists

Industry data on AI-assisted code (GitClear 2025, cURL bug-bounty shutdown 2025, arXiv 2510.03029):

- **Refactoring collapsed** from 25% to <10% of changes
- **Copy-paste surged** 8.3% → 12.3%; code duplication rose ~8x
- **Code-smell rates +42–85%** over human baselines
- **82% of AI PRs** use generic catch blocks that don't distinguish error types
- **76% miss timeouts** on external calls

None of this fails a typical CI lint. It just makes the codebase slowly unmaintainable. This skill is the lens for catching it before it ships.

The core insight: **reading cost > writing cost now**. The cost of writing code collapsed; the cost of reading it didn't. Code you can't quickly understand is slop, even if it works.

## How to Audit

When the user asks "review for AI slop", "audit code-quality taste", or "find AI patterns" — run through this skill's rules as a checklist against the changed files (PR diff) or full repo.

### Audit Step 1: Determine Scope

- If a PR diff is provided: audit only files changed in the diff
- If files are named: audit those
- If no scope: audit the whole repo, prioritized by recently-touched files (most likely AI output)

### Audit Step 2: Detect Stack

| Signal | Stack |
|--------|-------|
| `composer.json` + `artisan` | PHP / Laravel |
| `package.json` (with TypeScript/React deps) | Node / TypeScript / React |
| Both present | Laravel + Inertia + React |

### Audit Step 3: Run the Slop Checklist

For each item below, output:
- **CLEAN** — pattern not present (brief confirmation)
- **SUSPICIOUS** — present in small amounts; flag and discuss
- **INFLATED** — present extensively; verbose-but-functional; remediation recommended
- **CRITICAL** — extensive AI-fingerprint presence; full review needed before merge

#### Comments
- [ ] No comments that just narrate the code (`// create user` above `User::create(...)`)
- [ ] No empty docblocks (`/** Get user */` above `getUser()`)
- [ ] No placeholder comments left in (`// TODO: implement`, `// your code here`, `// implementation`)
- [ ] No closing-brace labels (`} // end function`, `} // end if block`)

#### Naming
- [ ] No generic placeholder names (`data`, `result`, `info`, `temp`, `helper`)
- [ ] No over-descriptive run-on names (`theUserWhoIsCurrentlyLoggedIn`)
- [ ] No suffix abuse (`*Helper` / `*Manager` / `*Util` / `*Wrapper` overused without justification)
- [ ] No type-in-name patterns (`userObject`, `resultArray`, `stringData`)

#### Over-engineering
- [ ] No interfaces with exactly one implementation (and no plan for a second)
- [ ] No single-method classes that should be top-level functions
- [ ] No wrapper functions called once that just delegate
- [ ] No new dependency added when an existing one does the same job

#### Defensive overdose
- [ ] No generic `catch (e) { console.error(...) }` blocks around code that can't throw
- [ ] No null checks for impossible nulls (after non-null assertions / type-guaranteed values)
- [ ] Real defensive concerns (timeouts on external calls, rate limits) ARE present

#### Test slop
- [ ] No tests that mock every dependency with no real behavioural assertion
- [ ] No "doesn't throw" tests that just call and check for exceptions
- [ ] No tests that mirror the implementation's logic (re-encoding rather than verifying)
- [ ] No snapshot tests standing in for behavioural assertions

#### Style fingerprints
- [ ] Some formatting drift exists (no codebase looks like every file ran through the most aggressive linter)
- [ ] No `as any` / `@ts-ignore` / `@ts-expect-error` sprinkled where inference is hard
- [ ] Repo has some `// HACK:` / `// XXX:` / 2am comments somewhere — codebases without scars are suspect
- [ ] No debug artifacts (`console.log`, `var_dump`, `dd()`, `dump()`) left in production code
- [ ] No `if (x) return true; else return false` / redundant type annotations on obvious literals

### Audit Step 4: Build the Slop Ledger

End the audit with a verdict table:

```
## Code Slop Ledger

| File | Verdict | Top findings | Suggested action |
|------|---------|--------------|------------------|
| app/Services/UserExportService.php | INFLATED | 12 narration comments; 3 closing-brace labels; `*Helper` overuse | Strip comments; rename Helper → split into functions |
| resources/js/Pages/Orders/Show.tsx | CRITICAL | 4 `as any`; mock-everything tests; useless wrapper; impossible null checks | Rewrite section; remove tests; revisit type model |
| app/Models/Order.php | CLEAN | — | — |

## Summary
- CLEAN: X files
- SUSPICIOUS: Y files
- INFLATED: Z files (top priority: …)
- CRITICAL: N files (rewrite before merge)
```

## When to Apply

Reference this skill when:
- Reviewing an AI-assisted PR before merge
- Auditing a repo that has accepted heavy AI-assisted contributions
- Onboarding a codebase and assessing whether it reads as human-maintained
- Hardening a team's code-review checklist against AI slop
- After a "vibe coding" sprint, before declaring features done
- Setting up CI gates for AI-output quality

## Step 1: Detect Project Stack

Most rules are stack-agnostic in concept, but examples and detection commands differ between PHP and TypeScript.

| Signal | Stack | Tooling |
|--------|-------|---------|
| `composer.json` | PHP / Laravel | `phpstan`, `phpcs`, `phpmd`, manual grep |
| `package.json` | Node / TS / React | `eslint`, `tsc --noEmit`, `knip`, manual grep |

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Comments | CRITICAL | `comments-` |
| 2 | Naming | CRITICAL | `naming-` |
| 3 | Over-engineering | HIGH | `over-eng-` |
| 4 | Defensive overdose | HIGH | `defensive-` |
| 5 | Test slop | HIGH | `test-` |
| 6 | Style fingerprints | MEDIUM | `style-` |

## Quick Reference

### 1. Comments (CRITICAL)

- `comments-narration` — Comments that just restate the code on the next line
- `comments-empty-docblocks` — Generic `/** Get the user */` over a typed `getUser()` signature
- `comments-placeholder` — `// TODO: implement`, `// your code here`, `// implementation`, `// helper function`
- `comments-closing-brace-labels` — `} // end function` / `} // end if block`

### 2. Naming (CRITICAL)

- `naming-generic-placeholders` — `data`, `result`, `info`, `temp`, `helper`, `value`
- `naming-over-descriptive` — `theUserWhoIsCurrentlyLoggedIn`, `calculateTotalAmountFromItemsList`
- `naming-suffix-abuse` — `*Helper` / `*Manager` / `*Util` / `*Wrapper` / `*Processor` overused
- `naming-type-in-name` — `userObject`, `resultArray`, `stringData`, `listOfItems`

### 3. Over-engineering (HIGH)

- `over-eng-premature-interface` — Interface with exactly one implementation and no second on the roadmap
- `over-eng-single-method-class` — Classes that exist solely to wrap one function
- `over-eng-useless-wrapper` — Wrapper called from exactly one place, just delegating
- `over-eng-dependency-creep` — New library when an existing dep already does the job

### 4. Defensive overdose (HIGH)

- `defensive-generic-catch` — `try { ... } catch (e) { console.error("error") }` everywhere
- `defensive-impossible-null` — Null checks after non-null assertions / type-guaranteed values
- `defensive-missing-real` — Defensive in the wrong places; missing timeouts/rate-limits where it matters

### 5. Test slop (HIGH)

- `test-mock-everything` — Mock for every dep; the test re-encodes the implementation, not the behaviour
- `test-doesnt-throw` — Tests that just call the function and assert no exception
- `test-mirror-implementation` — Tests whose logic mirrors the production code being tested
- `test-snapshot-abuse` — Snapshot tests replacing behavioural assertions

### 6. Style fingerprints (MEDIUM)

- `style-hyper-consistent` — No formatting drift anywhere; every file looks linter-perfect
- `style-as-any-escape` — `as any` / `@ts-ignore` / `@ts-expect-error` sprinkled where types are hard
- `style-no-hack-scars` — Codebase has zero `// HACK:` / `// XXX:` markers; no "geology"
- `style-debug-artifacts` — `console.log`, `var_dump`, `dd()`, `dump()` left in production paths
- `style-trivial-boilerplate` — `if (x) return true; else return false;`, redundant TS type annotations on obvious literals

## Essential Patterns

### The "would this pass code review?" filter

For each code chunk, ask:
1. **Could I cut a third of these comments and the code would be clearer?** → likely comment slop
2. **Do the variable names tell me what they hold, or just what type they are?** → likely naming slop
3. **Could this class be a function?** → likely over-engineering
4. **Does this catch block actually handle anything, or just log?** → likely defensive overdose
5. **Does this test fail if I break the function?** → if no, test slop
6. **Are there any `// HACK:` / `// XXX:` markers in the diff?** → if no, suspicious for AI

### Verdict bands (matched to AI-SLOP-Detector's scoring)

| Verdict | Meaning | Action |
|---|---|---|
| **CLEAN** | < 5% of lines flagged | Ship |
| **SUSPICIOUS** | 5–15% flagged | Review changes one more time |
| **INFLATED** | 15–30% flagged | Strip slop, split commits |
| **CRITICAL** | > 30% flagged | Rewrite section before merge |

## How to Use

Read individual rule files for detailed conventions and examples:

```
rules/comments-narration.md
rules/naming-generic-placeholders.md
rules/over-eng-premature-interface.md
rules/defensive-generic-catch.md
rules/test-mock-everything.md
rules/style-hyper-consistent.md
```

Each rule file contains:
- YAML frontmatter (title, impact, tags)
- Brief explanation of why the pattern is AI-fingerprint
- "Incorrect" example showing the slop
- "Correct" example showing the human-equivalent
- Detection guidance (grep / eslint / phpstan / heuristic)
- Reference link

## References

- GitClear — 2025 Code-Quality Trends Report (refactoring collapse, copy-paste surge)
- [arXiv 2510.03029 — Investigating the Smells of LLM-Generated Code](https://arxiv.org/abs/2510.03029)
- Addy Osmani — Comprehension Debt (O'Reilly Radar)
- Stack Overflow Blog — Eno Reyes Q&A on AI code quality

- [hardikpandya/stop-slop](https://github.com/hardikpandya/stop-slop) — sister project for prose slop
- [flamehaven01/AI-SLOP-Detector](https://github.com/flamehaven01/AI-SLOP-Detector) — Python AST scanner with 27 patterns

## Full Compiled Document

For the complete guide with all rules expanded: `AGENTS.md`
