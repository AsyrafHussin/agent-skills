---
name: technical-debt
description: Technical debt inventory, prioritization, and audit for PHP/Laravel (MySQL) and Node/TypeScript/React projects. Use when assessing code health, identifying refactoring candidates, planning debt paydown, or auditing a codebase for accumulated debt. Triggers on "audit technical debt", "find tech debt", "debt inventory", "what should we refactor first", or tasks involving code health, security debt, performance debt, data debt, observability debt, debt prioritization, or remediation planning.
license: MIT
metadata:
  author: agent-skills
  version: "1.0.0"
---

# Technical Debt

Technical debt audit and prioritization framework for **PHP/Laravel (MySQL) and Node/TypeScript/React** projects. Contains 42 rules across 10 categories covering code, security, design, dependency, test, performance, data, documentation, infrastructure, and process debt. Produces a ranked ledger (effort × impact) so teams know **what to fix first**, not just what's broken. Supports both **coding reference** and **audit mode** with PASS/FAIL/N/A output.

## Metadata

- **Version:** 1.0.0
- **Scope:** PHP / Laravel (MySQL) + Node / TypeScript / React
- **Rule Count:** 42 rules across 10 categories
- **License:** MIT

## How to Audit

When the user asks to "audit technical debt", "find tech debt", or "what should we refactor first" — run the checklist below against their codebase and produce a ranked debt ledger.

### Audit Step 1: Determine Scope

- If arguments provided (`$ARGUMENTS`): audit only those paths or modules
- If no arguments: audit the entire repository starting from the root

### Audit Step 2: Detect Project Stack

Inspect `composer.json` and `package.json` to determine which of the supported stacks (PHP/Laravel, Node/TypeScript/React, or both) is in use. Tooling commands (`composer outdated`, `npm outdated`, `phpstan`, `eslint`, `knip`, etc.) are chosen based on this detection.

### Audit Step 3: Run Debt Checklist

Work through every item below. For each, output:
- **PASS** — brief confirmation of what was verified
- **FAIL** — exact `file:line` (or command output), description of the debt, **effort estimate** (S/M/L), and **impact** (LOW/MED/HIGH/CRITICAL)
- **N/A** — if the check does not apply to this project

#### Code Debt
- [ ] No duplicated blocks > 30 lines across files
- [ ] No function exceeds 50 lines or cyclomatic complexity > 10
- [ ] No class exceeds 300 lines or has > 15 public methods (god class)
- [ ] No unreachable code, unused exports, or commented-out blocks left in source
- [ ] No magic numbers or unexplained string literals in business logic
- [ ] No function exceeds 4 parameters (use a parameter object)

#### Security Debt
- [ ] No secrets, API keys, or credentials committed to source (or git history)
- [ ] Every public endpoint validates input via a schema / FormRequest / DTO
- [ ] Passwords hashed with bcrypt or argon2id; no MD5/SHA-1
- [ ] Sessions have a bounded lifetime; auth-sensitive endpoints rate-limited
- [ ] Security headers present (CSP, HSTS, X-Content-Type-Options, Referrer-Policy)
- [ ] Authorization (not just authentication) enforced on every protected route

#### Design Debt
- [ ] No circular dependencies between modules/packages
- [ ] Layers respect direction (UI → service → repository, never reversed)
- [ ] No "shotgun surgery" patterns (one change forcing edits in 5+ files)
- [ ] Abstractions hide implementation details (no leaking framework types across boundaries)

#### Dependency Debt
- [ ] No dependencies more than 2 major versions behind
- [ ] No abandoned/unmaintained packages (no release in > 24 months)
- [ ] No known CVEs reported by `npm audit` / `composer audit` at HIGH or CRITICAL
- [ ] No unused dependencies in `package.json` / `composer.json`

#### Test Debt
- [ ] Critical paths have integration test coverage
- [ ] No skipped/disabled tests without linked issue or removal date
- [ ] No known flaky tests left in main branch
- [ ] Test suite runs in under 10 minutes (or has explicit budget documented)

#### Performance Debt
- [ ] No N+1 query patterns on list endpoints (query count constant per request)
- [ ] No list endpoint returns an unbounded result set (pagination present)
- [ ] Frontend initial bundle within budget (~200 KB gzip); route-level code splitting in place
- [ ] Expensive read paths (aggregations, external APIs, static configs) are cached at a sensible layer

#### Data Debt
- [ ] Migrations are the only source of schema changes; production matches migration history
- [ ] All foreign-key and frequently-queried columns are indexed
- [ ] No orphaned child records; FK constraints enforced

#### Documentation Debt
- [ ] README reflects current setup and dev workflow
- [ ] No stale comments contradicting the code they describe
- [ ] Public APIs / exported modules have docblocks or type hints

#### Infrastructure Debt
- [ ] Runtime versions (Node, PHP) are on supported (non-EOL) releases
- [ ] No deprecated framework APIs in use (e.g., Laravel `Route::get()` deprecations)
- [ ] Build runs cleanly with zero warnings
- [ ] Secrets stored in a manager (Vault / Doppler / cloud SM); no long-lived shared credentials
- [ ] Structured logs, error tracking, p95 latency dashboards, and SLO-based alerts in place

#### Process Debt
- [ ] No `TODO`/`FIXME`/`HACK` comments older than 6 months without owner or ticket
- [ ] No `@deprecated` markers without a removal date or replacement
- [ ] Debt is tracked somewhere visible (issue tracker label, debt register, ADR)
- [ ] Every path has an owner (CODEOWNERS file present and current)
- [ ] No feature flags at 100% rollout for more than 6 weeks without a removal plan

### Audit Step 4: Build the Debt Ledger

End the audit with a prioritized table:

```
## Technical Debt Ledger

| # | Category | Item | File / Location | Effort | Impact | Priority |
|---|----------|------|-----------------|--------|--------|----------|
| 1 | deps | jQuery 1.12 (8y old, 3 CVEs) | package.json:14 | L | CRITICAL | P0 |
| 2 | code | OrderService god class (820 lines) | app/Services/OrderService.php | M | HIGH | P1 |
| 3 | test | 12 disabled tests in auth/ | tests/Feature/Auth/* | S | HIGH | P1 |
...

## Summary
- **PASS:** X checks
- **FAIL:** Y checks
- **N/A:** Z checks
- **Top 3 to pay down first:** (list highest-priority items with rationale)
```

**Priority formula:** `P0 = CRITICAL impact`, `P1 = HIGH impact`, `P2 = MED`, `P3 = LOW`. Within a priority, sort by ascending effort (cheap wins first).

**Effort scale:**
- **S** = under a day
- **M** = 1–5 days
- **L** = more than a week (likely needs to be broken down)

---

## When to Apply

Reference these guidelines when:
- Running a tech-debt audit on a codebase
- Planning a refactoring sprint or debt-paydown initiative
- Reviewing a PR that introduces shortcuts (and deciding whether to accept them)
- Building a debt register or backlog category in your issue tracker
- Justifying engineering investment to non-engineering stakeholders
- Onboarding to a new codebase and assessing its health
- Writing an ADR (architecture decision record) for a debt-related decision

## Step 1: Detect Project Stack

**Always detect the stack before running tooling.** This skill targets PHP / Laravel (with MySQL) and Node / TypeScript / React projects; detection commands below assume one or both are present.

| Signal | Stack | Tooling |
|--------|-------|---------|
| `composer.json` present | PHP / Laravel | `composer outdated`, `composer audit`, `phpstan`, `phpcs`, `phpmd`, `deptrac` |
| `package.json` present | Node / JS / TS / React | `npm outdated`, `npm audit`, `eslint`, `tsc --noEmit`, `knip`, `madge` |
| MySQL connection available | Database | `EXPLAIN`, `sys.schema_tables_with_full_table_scans`, `sys.schema_unused_indexes`, `sys.statement_analysis` |
| any repo | Secrets scan | `gitleaks git`, `trufflehog` |

If both stacks are present (e.g., Laravel + Inertia + React), run audits for each.

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Code Debt | CRITICAL | `code-` |
| 2 | Security Debt | CRITICAL | `security-` |
| 3 | Design Debt | HIGH | `design-` |
| 4 | Dependency Debt | HIGH | `deps-` |
| 5 | Test Debt | HIGH | `test-` |
| 6 | Performance Debt | HIGH | `perf-` |
| 7 | Data Debt | HIGH | `data-` |
| 8 | Documentation Debt | MEDIUM | `docs-` |
| 9 | Infrastructure Debt | MEDIUM | `infra-` |
| 10 | Process Debt | MEDIUM | `process-` |

## Quick Reference

### 1. Code Debt (CRITICAL)

- `code-duplication` — Detect and consolidate duplicated logic
- `code-complexity` — Cyclomatic and cognitive complexity thresholds
- `code-long-functions` — Function and method length limits
- `code-god-classes` — Class size and responsibility limits
- `code-dead-code` — Unused code, unreachable branches, commented blocks
- `code-magic-numbers` — Hardcoded literals in business logic
- `code-long-parameter-lists` — Functions with too many positional params

### 2. Security Debt (CRITICAL)

- `security-secrets-in-code` — API keys, passwords, tokens in source / history
- `security-input-validation` — Endpoints accepting untrusted input without schemas
- `security-auth-hardening` — Outdated auth, missing MFA, missing security headers

### 3. Design Debt (HIGH)

- `design-tight-coupling` — Excessive direct dependencies between modules
- `design-circular-deps` — Cyclic imports / requires
- `design-leaky-abstractions` — Framework types crossing layer boundaries
- `design-shotgun-surgery` — Changes that touch many files at once

### 4. Dependency Debt (HIGH)

- `deps-outdated-versions` — Major versions behind on dependencies
- `deps-abandoned-packages` — Unmaintained / abandoned libraries
- `deps-security-advisories` — Known CVEs in installed dependencies
- `deps-unused-deps` — Declared but unused packages

### 5. Test Debt (HIGH)

- `test-coverage-gaps` — Critical paths without tests
- `test-flaky-tests` — Tests with non-deterministic outcomes
- `test-disabled-tests` — Skipped tests left in the suite
- `test-slow-tests` — Tests blocking fast feedback

### 6. Performance Debt (HIGH)

- `perf-n-plus-one` — Linear request → quadratic database load
- `perf-missing-pagination` — Unbounded result sets
- `perf-bundle-bloat` — Heavy / unsplit frontend bundles
- `perf-no-caching` — Missing cache layers on read-heavy paths

### 7. Data Debt (HIGH)

- `data-schema-drift` — Production schema diverges from migrations
- `data-missing-indexes` — Hot queries doing sequential scans
- `data-orphaned-records` — Referential integrity gaps

### 8. Documentation Debt (MEDIUM)

- `docs-stale-comments` — Comments contradicting current behavior
- `docs-outdated-architecture` — README/architecture docs out of date
- `docs-undocumented-api` — Public APIs without docblocks or types

### 9. Infrastructure Debt (MEDIUM)

- `infra-runtime-versions` — EOL or near-EOL language/runtime versions
- `infra-deprecated-apis` — Framework deprecations still in use
- `infra-build-warnings` — Build/compile warnings ignored
- `infra-secrets-management` — Long-lived credentials, plain env files, leaked logs
- `infra-monitoring-gaps` — Missing logs, metrics, traces, alerts, or SLOs

### 10. Process Debt (MEDIUM)

- `process-todo-fixme-aging` — Aging TODO/FIXME/HACK comments
- `process-deprecated-markers` — `@deprecated` without removal plan
- `process-debt-tracking` — Debt visible in a register / backlog
- `process-ownership-gaps` — Code without an owning team (CODEOWNERS)
- `process-feature-flags-lingering` — Stale feature flags polluting code paths

## Essential Patterns

### Debt Ledger Output Format

```
| # | Category | Item                          | Location              | Effort | Impact   | Priority |
|---|----------|-------------------------------|-----------------------|--------|----------|----------|
| 1 | deps     | guzzle 6.x (5y behind)        | composer.json:18      | M      | HIGH     | P1       |
| 2 | code     | InvoiceService dup logic      | app/Services/Invoice* | S      | MEDIUM   | P2       |
| 3 | test     | 8 skipped tests in checkout/  | tests/Feature/Checkout| S      | HIGH     | P1       |
```

### Effort × Impact Prioritization

```
              LOW       MEDIUM    HIGH      CRITICAL
S (<1d)   →   P3        P2        P1        P0
M (1-5d)  →   P3        P2        P1        P0
L (>1w)   →   P3        P3        P2        P1   (break down)
```

Cheap + high-impact items go first. Expensive items always get broken down before scheduling.

### Tooling Cheatsheet

```bash
# PHP / Laravel
composer outdated --direct          # list outdated direct deps
composer audit                      # known CVEs
vendor/bin/phpstan analyse          # static analysis
vendor/bin/phpmd app text cleancode # mess detector

# Node / TS
npm outdated                        # list outdated deps
npm audit                           # known CVEs
npx depcheck                        # unused deps
npx tsc --noEmit                    # type errors
npx eslint . --max-warnings 0       # lint warnings

# Cross-language
git log --since="6 months ago" --diff-filter=A -p | grep -E "TODO|FIXME|HACK"
cloc .                              # lines of code per language
```

## How to Use

Read individual rule files for detailed explanations and code examples:

```
rules/code-duplication.md
rules/design-circular-deps.md
rules/deps-outdated-versions.md
rules/test-flaky-tests.md
rules/process-todo-fixme-aging.md
```

Each rule file contains:
- YAML frontmatter with metadata (title, impact, tags)
- Brief explanation of why it matters
- How to detect (commands / patterns)
- Incorrect example with explanation
- Correct example or remediation strategy

## References

- [Martin Fowler — Technical Debt Quadrant](https://martinfowler.com/bliki/TechnicalDebtQuadrant.html)
- [Ward Cunningham — The Debt Metaphor](https://www.youtube.com/watch?v=pqeJFYwnkjE)
- [SonarQube — Technical Debt Model](https://docs.sonarsource.com/sonarqube/latest/user-guide/metric-definitions/)
- [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)
- [Snyk Open Source Vulnerability Database](https://security.snyk.io/)

## Full Compiled Document

For the complete guide with all rules expanded: `AGENTS.md`
