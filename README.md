# Agent Skills

A collection of skills for AI coding agents, designed for Claude Code.

Skills follow the [Agent Skills](https://agentskills.io/) format.

## Available Skills

### laravel-best-practices

Laravel 12 conventions and best practices. Contains 45+ rules across 7 categories for building scalable, maintainable Laravel applications.

**Use when:**
- Creating controllers, models, and services
- Writing migrations and database queries
- Implementing validation and form requests
- Building APIs with Laravel
- Structuring Laravel applications

**Categories covered:**
- Architecture & Structure (Critical)
- Eloquent & Database (Critical)
- Controllers & Routing (High)
- Validation & Requests (High)
- Security (High)
- Performance (Medium)
- API Design (Medium)

```bash
npx skills add AsyrafHussin/agent-skills --skill laravel-best-practices
```

---

### laravel-inertia-react

Laravel + Inertia.js + React patterns for building full-stack applications with a seamless SPA experience.

**Use when:**
- Building Laravel + Inertia.js + React applications
- Implementing Inertia forms and page components
- Sharing data between Laravel and React
- Handling Inertia redirects and flash messages

**Categories covered:**
- Page Components (Critical)
- Forms & Validation (High)
- Shared Data & Props (High)
- Navigation & Redirects (Medium)

```bash
npx skills add AsyrafHussin/agent-skills --skill laravel-inertia-react
```

---

### laravel-owasp-security

OWASP Top 10 security audit and secure coding guidelines for Laravel + React/Inertia.js. Dual-purpose — run a full audit or use as a secure coding reference.

**Use when:**
- "Run OWASP audit on my Laravel app"
- "Security review of my codebase"
- "Check my app for vulnerabilities"
- Writing auth, payment, or file upload logic
- Passing data from Laravel to Inertia props
- Using `dangerouslySetInnerHTML` in React

**Categories covered:**
- Broken Access Control (Critical) — A01:2021
- Cryptographic Failures (Critical) — A02:2021
- Injection — SQL & Mass Assignment (Critical) — A03:2021
- XSS & React/Inertia (High) — A03:2021
- CSRF Protection (High) — A08:2021
- Security Misconfiguration (High) — A05:2021
- Authentication & Rate Limiting (High) — A07:2021
- Inertia Data Exposure (High) — React/Inertia R2

```bash
npx skills add AsyrafHussin/agent-skills --skill laravel-owasp-security
```

---

### php-best-practices

Modern PHP 8.x patterns, PSR standards, and SOLID principles for clean, maintainable code.

**Use when:**
- Writing PHP classes, interfaces, and traits
- Applying SOLID principles
- Following PSR-12 coding standards
- Using PHP 8.x modern features

**Categories covered:**
- Type System (Critical)
- Modern Features (Critical)
- PSR Standards (High)
- SOLID Principles (High)
- Error Handling (High)
- Performance (Medium)
- Security (Critical)

```bash
npx skills add AsyrafHussin/agent-skills --skill php-best-practices
```

---

### react-vite-best-practices

React + Vite performance optimization guidelines. Contains 40+ rules across 8 categories.

**Use when:**
- Writing new React components
- Optimizing bundle size or load times
- Implementing data fetching
- Reviewing code for performance issues

**Categories covered:**
- Bundle Optimization (Critical)
- Component Performance (High)
- Data Fetching (High)
- Re-render Optimization (Medium)
- Code Splitting (Medium)

```bash
npx skills add AsyrafHussin/agent-skills --skill react-vite-best-practices
```

---

### typescript-react-patterns

TypeScript best practices and patterns for React applications.

**Use when:**
- Typing React components and hooks
- Working with generics in TypeScript
- Defining prop interfaces and types
- Reviewing TypeScript for type safety

**Categories covered:**
- Component Types (Critical)
- Generics (High)
- Utility Types (High)
- Type Narrowing (Medium)

```bash
npx skills add AsyrafHussin/agent-skills --skill typescript-react-patterns
```

---

### state-management

React state management patterns using React Query and Zustand.

**Use when:**
- Implementing server state with React Query
- Managing global state with Zustand
- Handling async data fetching and caching
- Syncing server and client state

**Categories covered:**
- React Query (Critical)
- Zustand (High)
- Caching Strategies (High)
- Optimistic Updates (Medium)

```bash
npx skills add AsyrafHussin/agent-skills --skill state-management
```

---

### tailwind-best-practices

Tailwind CSS patterns for responsive, dark mode-ready, and maintainable UIs.

**Use when:**
- Writing Tailwind utility classes
- Building responsive layouts
- Implementing dark mode
- Organizing component styles

**Categories covered:**
- Responsive Design (Critical)
- Dark Mode (High)
- Component Patterns (High)
- Performance (Medium)

```bash
npx skills add AsyrafHussin/agent-skills --skill tailwind-best-practices
```

---

### web-design-guidelines

UI/UX and accessibility guidelines. Audits code for 50+ rules covering accessibility, UX, and responsive design.

**Use when:**
- "Review my UI"
- "Check accessibility"
- "Audit design"
- "Review UX"

**Categories covered:**
- Accessibility (Critical)
- Forms & Validation (High)
- Typography (High)
- Responsive Design (Medium)
- Dark Mode & Theming (Medium)
- Performance (Medium)

```bash
npx skills add AsyrafHussin/agent-skills --skill web-design-guidelines
```

---

### clean-code-principles

SOLID, DRY, and clean code design patterns for maintainable software.

**Use when:**
- Reviewing code for code smells
- Applying SOLID principles
- Refactoring complex classes
- Designing class interfaces

**Categories covered:**
- SOLID Principles (Critical)
- DRY Patterns (High)
- Design Patterns (High)
- Naming Conventions (Medium)

```bash
npx skills add AsyrafHussin/agent-skills --skill clean-code-principles
```

---

### api-design-patterns

RESTful API design patterns for consistent, scalable APIs.

**Use when:**
- Designing REST API endpoints
- Structuring API responses and errors
- Implementing API versioning
- Building API resources and transformers

**Categories covered:**
- REST Conventions (Critical)
- Error Handling (High)
- Pagination (High)
- Versioning (Medium)

```bash
npx skills add AsyrafHussin/agent-skills --skill api-design-patterns
```

---

### git-workflow

Git workflow conventions for commits, branching, and pull requests.

**Use when:**
- Writing commit messages
- Creating branches and PRs
- Reviewing git history
- Following conventional commits

**Categories covered:**
- Commit Messages (Critical)
- Branching Strategy (High)
- Pull Requests (High)
- Code Review (Medium)

```bash
npx skills add AsyrafHussin/agent-skills --skill git-workflow
```

---

### testing-best-practices

Unit testing, integration testing, and TDD principles for reliable test suites.

**Use when:**
- Writing unit or feature tests
- Setting up test factories and mocks
- Reviewing test coverage
- Applying TDD principles

**Categories covered:**
- Test Structure (Critical)
- Test Isolation (Critical)
- Mocking (High)
- Coverage (Medium)

```bash
npx skills add AsyrafHussin/agent-skills --skill testing-best-practices
```

---

## Installation

Install all skills at once:

```bash
npx skills add AsyrafHussin/agent-skills
```

## Usage

Skills are automatically available once installed. The agent will use them when relevant tasks are detected.

**Examples:**
```
Run OWASP security audit on my Laravel app
```
```
Review this Laravel controller for best practices
```
```
Check my React component for performance issues
```

## License

MIT
