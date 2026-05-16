# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. Code Debt (code)

**Impact:** CRITICAL
**Description:** Debt within the code itself — duplication, complex methods, oversized classes, dead code, magic literals, and long parameter lists. These are the most direct drivers of bug density and slow feature work, and they compound fastest as the codebase grows.

## 2. Security Debt (security)

**Impact:** CRITICAL
**Description:** Accumulated security gaps — secrets in code, missing input validation, outdated auth defaults, and missing hardening. Security debt is the category most likely to become a *public* problem; what's tolerable as a backlog item today is tomorrow's breach disclosure.

## 3. Design Debt (design)

**Impact:** HIGH
**Description:** Structural debt across modules — tight coupling, circular dependencies, leaky abstractions, and changes that ripple through many files. Design debt makes refactoring expensive and concentrates risk in small changes.

## 4. Dependency Debt (deps)

**Impact:** HIGH
**Description:** Debt in third-party packages — outdated versions, abandoned libraries, known CVEs, and unused dependencies. Dependency debt is the cheapest debt to detect (tooling does it for you) and the most dangerous to ignore (it grows by itself even when you don't touch the code).

## 5. Test Debt (test)

**Impact:** HIGH
**Description:** Gaps in test coverage, flaky or disabled tests, and slow suites that erode confidence in CI. Test debt directly slows the team — a flaky or slow suite costs every engineer every day.

## 6. Performance Debt (perf)

**Impact:** HIGH
**Description:** N+1 queries, unbounded result sets, and bundle bloat. Performance debt looks fine in development and explodes proportionally to your most successful customer. It is the category most likely to convert directly into lost revenue.

## 7. Data Debt (data)

**Impact:** HIGH
**Description:** Schema drift, missing indexes, and referential-integrity gaps. Data debt is the slowest to detect and the hardest to fix — every migration after the drift is a roll of the dice, and orphaned records propagate into reports nobody trusts.

## 8. Documentation Debt (docs)

**Impact:** MEDIUM
**Description:** Stale comments, outdated READMEs, and undocumented public APIs. Documentation debt is invisible until onboarding or incident response, where it suddenly becomes the bottleneck.

## 9. Infrastructure Debt (infra)

**Impact:** MEDIUM
**Description:** EOL runtimes, deprecated framework APIs, accumulated build warnings, insecure secrets handling, and observability gaps. Infrastructure debt has hard deadlines (CVEs, vendor EOL dates) and is non-negotiable once they hit.

## 10. Process Debt (process)

**Impact:** MEDIUM
**Description:** Aging TODO/FIXME comments, `@deprecated` markers without removal plans, untracked debt, and code without owners. Process debt is about visibility and accountability — debt you can't see, or that nobody owns, cannot be prioritized.
