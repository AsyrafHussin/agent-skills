# Agent Skills Repository — Operational Guide

This repository contains comprehensive skills for AI coding agents, optimized for:
- React + Vite
- React + TypeScript
- Laravel 13 + Inertia.js + React
- Tailwind CSS
- React Query + Zustand
- PHP 8.x
- Clean Code & Design Patterns

---

## Rule Priority Model

When rules from different skills conflict, resolve them in this order (highest wins):

| Priority | Category | Examples |
|----------|----------|---------|
| 1 | **Safety / Security** | OWASP, injection, authentication, encryption |
| 2 | **System / Developer constraints** | PHP version, framework version, detected stack |
| 3 | **User request** | Explicit instructions from the user |
| 4 | **Project workflow rules** | Git workflow, PR conventions, testing requirements |
| 5 | **Style / Formatting preferences** | Naming, spacing, comment style |

**When in doubt:** ask the user rather than guess.

---

## Step 1 — Detect Context

Before applying any skill, identify the project stack. Check the following files:

### PHP / Laravel
```bash
# PHP version
grep '"php"' composer.json          # e.g. "^8.3"
php -v                               # runtime version

# Laravel version
grep '"laravel/framework"' composer.json

# Testing framework
grep '"pestphp/pest"' composer.json  # Pest present → use Pest syntax
grep '"phpunit/phpunit"' composer.json
# tests/Pest.php exists → Pest is configured
```

### Frontend
```bash
# React / Vite
grep '"react"' package.json
grep '"vite"' package.json

# Inertia.js
grep '"@inertiajs/react"' package.json
grep 'inertia-laravel' composer.json

# State management
grep '"@tanstack/react-query"' package.json   # React Query
grep '"zustand"' package.json                  # Zustand
```

### Testing Framework Decision

| Condition | Action |
|-----------|--------|
| `pestphp/pest` in `composer.json` | Use **Pest** syntax |
| Only `phpunit/phpunit` (no Pest) | Use **PHPUnit** syntax |
| Both present | **Pest takes priority** |
| `tests/Pest.php` exists | Use **Pest** syntax |
| Neither detected | Ask the user: *"Does this project use Pest PHP or PHPUnit?"* |

---

## Step 2 — Activate Relevant Skills

Load only the skills that match the detected stack and user request. Use the table below.

### Skill Activation Matrix

| Skill | Activate when... |
|-------|-----------------|
| `laravel-best-practices` | Writing or reviewing PHP / Laravel code |
| `laravel-inertia-react` | Project has `@inertiajs/react` **and** Laravel |
| `laravel-testing` | Writing tests in a Laravel project (Pest or PHPUnit) |
| `laravel-owasp-security` | Security audit, or before any auth / payment code is merged |
| `laravel-database-optimization` | Query performance, N+1 detection, Eloquent optimization |
| `laravel-ai-sdk` | Code uses `Laravel\Ai` namespace or AI SDK features |
| `laravel-mcp` | Building or consuming MCP servers |
| `php-best-practices` | Any PHP code (applies on top of Laravel skill) |
| `operational-prompting` | Designing AGENTS.md, Copilot/Codex instructions, task contracts, or portable agent manifests |
| `react-vite-best-practices` | React + Vite project (build config, code splitting) |
| `typescript-react-patterns` | TypeScript files in a React project |
| `tailwind-best-practices` | Tailwind CSS classes appear in the code |
| `state-management` | React Query, Zustand, or complex client state |
| `web-design-guidelines` | UI/UX work, accessibility, responsive layout |
| `api-design-patterns` | Designing or reviewing REST API endpoints |
| `clean-code-principles` | Architecture review, refactoring, design decisions |
| `code-review-architecture` | Targeted architecture review or multi-lens design analysis |
| `code-review-error-handling` | Reviewing failure paths, retries, timeouts, or resilience behavior |
| `code-review-performance` | Investigating bottlenecks, scalability risks, or resource-heavy diffs |
| `code-review-security` | Reviewing vulnerabilities, validation, auth, or secret handling |
| `code-review-simplicity` | Reviewing readability, complexity, duplication, or dead-range logic |
| `code-review-type-safety` | Reviewing typings, schema alignment, and unsafe coercions |
| `testing-best-practices` | Any test code (language-agnostic patterns) |
| `git-workflow` | Reviewing commits, branch names, PR descriptions |
| `seo-best-practices` | Public-facing pages, meta tags, structured data |

Multiple skills may be active at once. Apply them in the priority order defined above.

---

## Step 3 — Apply Rules in Priority Order

1. **CRITICAL** rules always apply first, regardless of skill.
2. **HIGH** rules apply next.
3. **MEDIUM** and **LOW** rules fill in the remaining gaps.
4. When two rules at the same impact level conflict, the higher-priority skill wins (see Rule Priority Model above).

Each skill's `AGENTS.md` defines rule categories tagged **CRITICAL**, **HIGH**, **MEDIUM**, or **LOW**. Refer to the relevant file in `skills/<skill-name>/AGENTS.md`.

---

## Step 4 — Generate Output

### Code Output
- Use only language features available in the detected version (e.g. PHP 8.3 features only if `php: ^8.3`).
- Prefer correct code over commented code — show what to do, not what not to do, unless a Bad/Good contrast is explicitly requested.
- Never expose secrets, credentials, `.env` values, or stack traces in output.

### Review Output
- Lead with the most critical issues.
- Use the impact tags: **CRITICAL**, **HIGH**, **MEDIUM**, **LOW**.
- Provide a concrete fix for every issue raised.
- Do not flag trade-offs or style preferences as defects.

### Audit / Checklist Output
Follow the output format defined in the activated skill (e.g. `laravel-owasp-security` uses PASS / FAIL / N/A per line).

---

## Step 5 — Agent Self-Check

Before finalizing any response, verify:

- [ ] Detected PHP / framework version and applied only compatible features
- [ ] Applied CRITICAL rules before HIGH, HIGH before MEDIUM
- [ ] No security information (keys, tokens, stack traces) included in output
- [ ] Every issue raised has a concrete fix
- [ ] Conflicts between skills resolved using the priority model
- [ ] If stack could not be detected, asked the user rather than assumed

---

## Skills Overview

| Skill | Rules | Priority Focus |
|-------|-------|----------------|
| api-design-patterns | 38 | REST, error handling, pagination |
| clean-code-principles | 23 | SOLID, DRY, design patterns |
| code-review-architecture | 8 | Coupling, boundaries, rollback-safe design |
| code-review-error-handling | 8 | Timeouts, retries, cleanup, observability |
| code-review-performance | 7 | Cost, queries, caching, concurrency |
| code-review-security | 6 | Injection, auth, validation, data protection |
| code-review-simplicity | 8 | Readability, complexity, duplication, bounds |
| code-review-type-safety | 4 | Typing, runtime validation, generics |
| git-workflow | 31 | Commits, branching, PRs |
| laravel-ai-sdk | 17 | Agents, tools, embeddings, testing |
| laravel-best-practices | 31 | Architecture, Eloquent, security |
| laravel-database-optimization | 33 | Eloquent queries, indexing, caching, N+1 |
| laravel-inertia-react | 24 | Page components, forms |
| laravel-mcp | 7 | MCP servers, tools, prompts, resources |
| laravel-owasp-security | 8 | OWASP Top 10, secure coding, auth |
| laravel-testing | 24 | Pest PHP 4 & PHPUnit 12, HTTP tests, factories, faking, AI SDK |
| operational-prompting | 8 | Repo-owned instructions, validation, portable skills |
| php-best-practices | 56 | PHP 8.x, static analysis, design |
| prd-writing | 25 | PRD workflow, feature specs, requirements |
| react-vite-best-practices | 23 | Build optimization, code splitting |
| seo-best-practices | 31 | Meta tags, structured data, Core Web Vitals, SEO audit |
| state-management | 26 | React Query, Zustand |
| tailwind-best-practices | 29 | Responsive, dark mode |
| testing-best-practices | 34 | Unit tests, mocking, coverage |
| typescript-react-patterns | 33 | Type safety, generics |
| web-design-guidelines | 23 | Accessibility, UX, WCAG audit |

Each skill lives in `skills/<skill-name>/` and contains:
- `SKILL.md` — Main skill definition with YAML frontmatter
- `AGENTS.md` — Operational rules and examples for agents, including the shared operational contract
- `rules/` — Individual rule files (loaded on demand)
- `metadata.json` — Structured metadata, canonical `skill` info, and `meta` compatibility fields

When a skill keeps legacy top-level metadata for compatibility, treat the nested `skill` block as the canonical identity block and `meta` as the canonical lifecycle/compatibility block.

---

## Contributing

When adding new rules:
1. Follow the existing rule template in `rules/_template.md`
2. Include incorrect and correct code examples
3. Explain the "why" not just the "what"
4. Add appropriate priority level
