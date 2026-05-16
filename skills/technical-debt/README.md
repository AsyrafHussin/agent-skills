# Technical Debt

Technical debt inventory, prioritization, and audit framework for **PHP/Laravel (MySQL) and Node/TypeScript/React** projects. Produces a ranked debt ledger (effort × impact) so teams know what to pay down first — not just what's broken. Supports both coding reference and audit mode with PASS/FAIL/N/A output.

**Version:** 1.0.0

## Overview

- Technical debt audit with PASS/FAIL/N/A checklist output
- Ranked debt ledger sorted by effort × impact (P0–P3)
- Code debt detection (duplication, complexity, god classes, dead code, magic numbers)
- Security debt (secrets, validation, auth hardening)
- Design debt (coupling, circular deps, leaky abstractions)
- Dependency debt (outdated, abandoned, CVEs, unused)
- Test debt (coverage gaps, flaky tests, disabled tests, slow suites)
- Performance debt (N+1 queries, unbounded results, bundle bloat, missing caching)
- Data debt (schema drift, missing indexes, orphaned records)
- Infrastructure debt (EOL runtimes, secrets management, observability)
- Process debt (aging TODOs, deprecated markers, ownership, feature flags)
- 42 rules across 10 categories

## Categories

### 1. Code Debt (CRITICAL)
Duplication, complexity, long functions, god classes, dead code, magic numbers, long parameter lists.

### 2. Security Debt (CRITICAL)
Secrets in source code, missing input validation, outdated auth and missing hardening (CSP, HSTS, rate limits, MFA).

### 3. Design Debt (HIGH)
Tight coupling, circular dependencies, leaky abstractions, shotgun surgery.

### 4. Dependency Debt (HIGH)
Outdated versions, abandoned packages, security advisories, unused deps.

### 5. Test Debt (HIGH)
Coverage gaps, flaky tests, disabled tests, slow tests.

### 6. Performance Debt (HIGH)
N+1 queries, unbounded result sets, frontend bundle bloat, missing caching opportunities.

### 7. Data Debt (HIGH)
Database schema drift, missing indexes on hot queries, orphaned records / referential gaps.

### 8. Documentation Debt (MEDIUM)
Stale comments, outdated architecture docs, undocumented public APIs.

### 9. Infrastructure Debt (MEDIUM)
EOL runtime versions, deprecated framework APIs, build warnings, secrets-management practices, observability and monitoring gaps.

### 10. Process Debt (MEDIUM)
Aging TODO/FIXME comments, `@deprecated` markers without removal plans, untracked debt, code without owners (CODEOWNERS), lingering feature flags.

## Usage

```
Audit technical debt in this project
What should we refactor first?
Build a debt ledger for the checkout module
Find the top 5 highest-impact tech debt items
Review this PR for new technical debt
Audit security debt: secrets, input validation, auth hardening
Find N+1 queries and missing indexes
```

## References

- [Martin Fowler — Technical Debt Quadrant](https://martinfowler.com/bliki/TechnicalDebtQuadrant.html)
- [Ward Cunningham — The Debt Metaphor](https://www.youtube.com/watch?v=pqeJFYwnkjE)
- [SonarQube — Technical Debt Model](https://docs.sonarsource.com/sonarqube/latest/user-guide/metric-definitions/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Snyk Vulnerability Database](https://security.snyk.io/)
- [Use the Index, Luke](https://use-the-index-luke.com/)
- [gitleaks](https://github.com/gitleaks/gitleaks)
