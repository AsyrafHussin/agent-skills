# Agent Skills

![Release](https://img.shields.io/github/v/release/AsyrafHussin/agent-skills?style=flat-square) ![Stars](https://img.shields.io/github/stars/AsyrafHussin/agent-skills?style=flat-square) ![License](https://img.shields.io/github/license/AsyrafHussin/agent-skills?style=flat-square)

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

Laravel 13 conventions and architecture. Contains 31 rules across 7 categories.

**Example:** `Review this Laravel controller for best practices`

```bash
npx skills add AsyrafHussin/agent-skills --skill laravel-best-practices
```

---

### [laravel-inertia-react](skills/laravel-inertia-react)

Laravel 13 + Inertia.js + React 18 full-stack patterns. Covers page components, form handling, layouts, file uploads, and real-time features.

**Example:** `How do I share data from Laravel to a React component with Inertia?`

```bash
npx skills add AsyrafHussin/agent-skills --skill laravel-inertia-react
```

---

### [laravel-testing](skills/laravel-testing)

Laravel 13 testing with Pest PHP 4 and PHPUnit 12. Contains 24 rules across 6 categories covering HTTP feature tests, model factories, database assertions, facade faking (Mail, Queue, Notification, Event, Storage, AI SDK), authentication testing, and test organisation patterns. Detects Pest or PHPUnit automatically from `composer.json`.

**Example:** `Write feature tests for this Laravel controller`

```bash
npx skills add AsyrafHussin/agent-skills --skill laravel-testing
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

React + Vite performance optimization. Contains 23 rules across 6 categories covering build config (OXC minification, tree shaking), code splitting (React.lazy, Suspense), asset handling, environment config, and bundle analysis.

**Example:** `Review this React component for performance issues`

```bash
npx skills add AsyrafHussin/agent-skills --skill react-vite-best-practices
```

---

### [typescript-react-patterns](skills/typescript-react-patterns)

Type-safe React with TypeScript. Contains 33 rules across 7 categories covering component typing, hooks, event handling, refs, generics, context, and utility types.

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

WCAG accessibility, semantic HTML, keyboard navigation, forms, and performance. Contains 23 rules across 4 categories covering accessibility (WCAG 2.2), form patterns, reduced motion, and layout stability.

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

RESTful API design patterns. Contains 38 rules across 7 categories covering resource design, error handling, security, pagination, versioning, response format, and OpenAPI documentation.

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

### [prd-writing](skills/prd-writing)

Step-by-step workflow for writing Product Requirements Documents. 6-step process: assess project state, ask clarifying questions (with lettered options), draft using 12-section template, review, revise, and save. Contains 25 rules across 7 categories as supporting knowledge.

**Example:** `Write a PRD for this feature`

```bash
npx skills add AsyrafHussin/agent-skills --skill prd-writing
```

---

### [testing-best-practices](skills/testing-best-practices)

Unit, integration, and E2E testing with TypeScript/Vitest. Contains 34 rules across 7 categories covering test structure (AAA), isolation, assertions, test data factories, mocking, coverage strategy, and test performance.

**Example:** `Write tests for this Laravel service class`

```bash
npx skills add AsyrafHussin/agent-skills --skill testing-best-practices
```

---

### [seo-best-practices](skills/seo-best-practices)

SEO patterns for Laravel Blade and Laravel + Inertia.js + React. Contains 31 rules across 8 categories covering Core Web Vitals, meta tags, structured data (JSON-LD, @graph, FAQ schema), Open Graph, performance, and mobile-first indexing. Auto-detects project type to apply the right rules.

**Example:** `Review my page for SEO issues`

```bash
npx skills add AsyrafHussin/agent-skills --skill seo-best-practices
```

---

### [laravel-database-optimization](skills/laravel-database-optimization)

Laravel 13 database optimization. Contains 33 rules across 9 categories covering N+1 prevention, indexing strategies, Eloquent performance, Redis caching, cursor pagination, transactions, zero-downtime migrations, naming conventions, and query debugging.

**Example:** `Optimize the database queries in this controller`

```bash
npx skills add AsyrafHussin/agent-skills --skill laravel-database-optimization
```

---

### [laravel-ai-sdk](skills/laravel-ai-sdk)

Laravel AI SDK for building AI-powered features. Contains 17 rules across 7 categories covering agents, tools, structured output, streaming, images, audio, embeddings, vector stores, and testing with fakes.

**Example:** `Create an AI agent that analyzes sales transcripts`

```bash
npx skills add AsyrafHussin/agent-skills --skill laravel-ai-sdk
```

---

### [laravel-mcp](skills/laravel-mcp)

Laravel MCP server development. Contains 7 rules across 5 categories covering server creation, tools with schemas, prompts, resources, authentication (OAuth/Sanctum), and testing.

**Example:** `Create an MCP tool that exposes weather data to AI clients`

```bash
npx skills add AsyrafHussin/agent-skills --skill laravel-mcp
```

---

## License

[MIT](LICENSE)
