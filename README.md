# Agent Skills

A collection of skills for AI coding agents. Works with Claude Code, Cursor, Codex, Windsurf, and [40+ agents](https://github.com/vercel-labs/skills#supported-agents).

Skills follow the [Agent Skills](https://agentskills.io/) specification. Discover more at [skills.sh](https://skills.sh).

## Installation

### Using npx (recommended)

```bash
# Install all skills
npx skills add AsyrafHussin/agent-skills

# Install a specific skill
npx skills add AsyrafHussin/agent-skills --skill laravel-best-practices

# Install multiple skills
npx skills add AsyrafHussin/agent-skills --skill laravel-best-practices --skill seo-best-practices

# Install globally (available across all projects)
npx skills add AsyrafHussin/agent-skills -g

# Install to a specific agent
npx skills add AsyrafHussin/agent-skills -a claude-code

# List available skills without installing
npx skills add AsyrafHussin/agent-skills --list
```

| Scope | Flag | Location | Use Case |
|-------|------|----------|----------|
| Project | (default) | `.claude/skills/` | Shared with team via git |
| Global | `-g` | `~/.claude/skills/` | Available across all projects |

### Manual installation

```bash
git clone https://github.com/AsyrafHussin/agent-skills.git

# Copy all skills
cp -r agent-skills/skills/* .claude/skills/

# Or copy a single skill
cp -r agent-skills/skills/laravel-best-practices .claude/skills/
```

Each skill contains `SKILL.md`, `AGENTS.md`, `rules/`, and `metadata.json`.

### Managing skills

```bash
npx skills list              # List installed skills
npx skills find laravel      # Search for skills
npx skills remove            # Remove installed skills
npx skills update            # Update all skills to latest
```

## Available Skills

### [laravel-best-practices](skills/laravel-best-practices)

Laravel 12 conventions and architecture. Contains 29 rules across 7 categories.

**Example:** `Review this Laravel controller for best practices`

```bash
npx skills add AsyrafHussin/agent-skills --skill laravel-best-practices
```

---

### [laravel-inertia-react](skills/laravel-inertia-react)

Laravel 12 + Inertia.js + React 18 full-stack patterns. Covers page components, form handling, layouts, file uploads, and real-time features.

**Example:** `How do I share data from Laravel to a React component with Inertia?`

```bash
npx skills add AsyrafHussin/agent-skills --skill laravel-inertia-react
```

---

### [laravel-owasp-security](skills/laravel-owasp-security)

OWASP Top 10 security audit and secure coding guidelines for Laravel + React/Inertia.js.

**Example:** `Run OWASP security audit on my Laravel app`

```bash
npx skills add AsyrafHussin/agent-skills --skill laravel-owasp-security
```

---

### [php-best-practices](skills/php-best-practices)

Modern PHP 8.0–8.5 patterns, type system, PSR standards, and SOLID principles. Contains 51 rules for secure, maintainable code.

**Example:** `Review my PHP class for SOLID principles`

```bash
npx skills add AsyrafHussin/agent-skills --skill php-best-practices
```

---

### [react-vite-best-practices](skills/react-vite-best-practices)

React + Vite performance optimization. Contains 27 rules across 8 categories covering build configuration, code splitting, bundle analysis, HMR, and asset handling.

**Example:** `Review this React component for performance issues`

```bash
npx skills add AsyrafHussin/agent-skills --skill react-vite-best-practices
```

---

### [typescript-react-patterns](skills/typescript-react-patterns)

Type-safe React with TypeScript 5+. Covers component props, hooks, event handling, refs, generics, and context patterns.

**Example:** `How do I properly type this React component with TypeScript?`

```bash
npx skills add AsyrafHussin/agent-skills --skill typescript-react-patterns
```

---

### [state-management](skills/state-management)

Server state with TanStack Query v5 and client state with Zustand v5. Covers caching, mutations, optimistic updates, and devtools.

**Example:** `How do I manage server state with React Query?`

```bash
npx skills add AsyrafHussin/agent-skills --skill state-management
```

---

### [tailwind-best-practices](skills/tailwind-best-practices)

Tailwind CSS v3.4+ and v4 best practices. Contains 29 rules across 8 categories covering responsive design, dark mode, component patterns, configuration, and v3-to-v4 migration.

**Example:** `Review my Tailwind classes for best practices`

```bash
npx skills add AsyrafHussin/agent-skills --skill tailwind-best-practices
```

---

### [web-design-guidelines](skills/web-design-guidelines)

WCAG accessibility, semantic HTML, keyboard navigation, color contrast, and motion preferences. Contains 21 rules across 8 categories.

**Example:** `Review my UI for accessibility and UX issues`

```bash
npx skills add AsyrafHussin/agent-skills --skill web-design-guidelines
```

---

### [clean-code-principles](skills/clean-code-principles)

Language-agnostic SOLID, DRY, KISS principles and design patterns (Factory, Strategy, Repository). Contains 23 rules for maintainable software.

**Example:** `Review this class for code smells and SOLID violations`

```bash
npx skills add AsyrafHussin/agent-skills --skill clean-code-principles
```

---

### [api-design-patterns](skills/api-design-patterns)

RESTful API design: HTTP methods, error handling, pagination, versioning, rate limiting, and security patterns.

**Example:** `Review my API endpoints for REST best practices`

```bash
npx skills add AsyrafHussin/agent-skills --skill api-design-patterns
```

---

### [git-workflow](skills/git-workflow)

Git workflow conventions for commits, branching, pull requests, and history management. Contains 31 rules covering conventional commits, git hooks, workflow strategies (GitHub Flow, GitFlow, Trunk-Based), monorepo workflows, git worktree, and .gitignore best practices.

**Example:** `Review my git workflow and suggest improvements`

```bash
npx skills add AsyrafHussin/agent-skills --skill git-workflow
```

---

### [testing-best-practices](skills/testing-best-practices)

Unit, integration, and E2E testing. Covers AAA pattern, test isolation, mocking, test data factories, coverage strategy, and TDD principles.

**Example:** `Write tests for this Laravel service class`

```bash
npx skills add AsyrafHussin/agent-skills --skill testing-best-practices
```

---

### [seo-best-practices](skills/seo-best-practices)

SEO patterns for Laravel Blade and Laravel + Inertia.js + React. Contains 28 rules across 8 categories covering Core Web Vitals, meta tags, structured data (JSON-LD), Open Graph, performance, and mobile-first indexing. Auto-detects project type to apply the right rules.

**Example:** `Review my page for SEO issues`

```bash
npx skills add AsyrafHussin/agent-skills --skill seo-best-practices
```

---

### [laravel-database-optimization](skills/laravel-database-optimization)

Laravel 12 database optimization. Contains 33 rules across 9 categories covering N+1 prevention, indexing strategies, Eloquent performance, Redis caching, cursor pagination, transactions, zero-downtime migrations, naming conventions, and query debugging.

**Example:** `Optimize the database queries in this controller`

```bash
npx skills add AsyrafHussin/agent-skills --skill laravel-database-optimization
```

---

## License

MIT
