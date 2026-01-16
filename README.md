# Agent Skills

A collection of comprehensive skills for AI coding agents, tailored for modern full-stack development with React, TypeScript, Laravel, and Tailwind CSS.

Skills follow the [Agent Skills](https://agentskills.io/) format and are designed for use with Claude Code.

## Available Skills

### react-vite-best-practices

React and Vite performance optimization guidelines. Contains 40+ rules across 8 categories, prioritized by impact.

**Use when:**
- Writing new React components with Vite
- Optimizing build configuration
- Implementing code splitting and lazy loading
- Reviewing code for performance issues

**Categories covered:**
- Build Optimization (Critical)
- Code Splitting (Critical)
- Development Performance (High)
- Asset Handling (High)
- Environment Configuration (Medium)
- HMR Optimization (Medium)
- Bundle Analysis (Low-Medium)

---

### typescript-react-patterns

TypeScript best practices for React development. Contains 35+ rules for type-safe React applications.

**Use when:**
- Writing typed React components
- Implementing custom hooks with TypeScript
- Handling events and refs with proper types
- Building generic, reusable components

**Categories covered:**
- Component Typing (Critical)
- Hook Typing (Critical)
- Event Handling (High)
- Generic Components (High)
- Context & State (Medium)
- Utility Types (Medium)
- Advanced Patterns (Low)

---

### laravel-best-practices

Laravel 12 conventions and best practices. Contains 45+ rules for building scalable Laravel applications.

**Use when:**
- Creating controllers, models, and services
- Writing migrations and database queries
- Implementing validation and form requests
- Structuring Laravel applications

**Categories covered:**
- Architecture & Structure (Critical)
- Eloquent & Database (Critical)
- Controllers & Routing (High)
- Validation & Requests (High)
- Security (High)
- Performance (Medium)
- Testing (Medium)

---

### laravel-inertia-react

Laravel + Inertia.js + React integration patterns. Contains 30+ rules for seamless full-stack development.

**Use when:**
- Building Inertia.js page components
- Handling forms with useForm hook
- Managing shared data and authentication
- Implementing persistent layouts

**Categories covered:**
- Page Components (Critical)
- Forms & Validation (Critical)
- Navigation & Links (High)
- Shared Data (High)
- Layouts (Medium)
- File Uploads (Medium)
- Advanced Patterns (Low)

---

### tailwind-best-practices

Tailwind CSS patterns and conventions. Contains 35+ rules for consistent, maintainable styling.

**Use when:**
- Writing responsive designs
- Implementing dark mode
- Creating reusable component styles
- Optimizing Tailwind configuration

**Categories covered:**
- Responsive Design (Critical)
- Dark Mode (Critical)
- Component Patterns (High)
- Custom Configuration (High)
- Spacing & Typography (Medium)
- Animation (Medium)
- Performance (Low)

---

### state-management

React Query + Zustand patterns for state management. Contains 40+ rules for efficient data fetching and client state.

**Use when:**
- Implementing data fetching with React Query
- Managing client state with Zustand
- Handling caching and invalidation
- Building optimistic updates

**Categories covered:**
- React Query Basics (Critical)
- Zustand Store Patterns (Critical)
- Caching & Invalidation (High)
- Mutations & Updates (High)
- Optimistic Updates (Medium)
- DevTools & Debugging (Medium)
- Advanced Patterns (Low)

---

### web-design-guidelines

UI/UX best practices and accessibility guidelines. Contains 50+ rules for building inclusive, performant interfaces.

**Use when:**
- Reviewing UI for accessibility
- Implementing forms and interactions
- Optimizing performance
- Ensuring cross-browser compatibility

**Categories covered:**
- Accessibility (Critical)
- Forms & Validation (Critical)
- Performance (High)
- Animation & Motion (High)
- Typography (Medium)
- Touch & Interaction (Medium)
- Internationalization (Low)

---

### php-best-practices

Modern PHP 8.x patterns, PSR standards, and SOLID principles. Contains 45+ rules for clean, maintainable PHP code.

**Use when:**
- Writing or reviewing PHP code
- Using PHP 8.x modern features
- Implementing classes and interfaces
- Following PSR standards

**Categories covered:**
- Type System (Critical)
- Modern Features (Critical)
- PSR Standards (High)
- SOLID Principles (High)
- Error Handling (High)
- Performance (Medium)
- Security (Critical)

---

### clean-code-principles

SOLID principles, design patterns, and clean code fundamentals. Language-agnostic guidelines for writing maintainable software.

**Use when:**
- Designing new features or systems
- Reviewing code architecture
- Refactoring existing code
- Discussing design decisions

**Categories covered:**
- SOLID Principles (Critical)
- Core Principles (Critical)
- Design Patterns (High)
- Code Organization (High)
- Naming & Readability (Medium)
- Functions & Methods (Medium)
- Comments & Documentation (Low)

---

### api-design-patterns

RESTful API design principles, error handling, pagination, and versioning. Guidelines for building consistent, developer-friendly APIs.

**Use when:**
- Designing new API endpoints
- Reviewing existing API structure
- Implementing error handling
- Setting up pagination/filtering

**Categories covered:**
- Resource Design (Critical)
- Error Handling (Critical)
- Security (Critical)
- Pagination & Filtering (High)
- Versioning (High)
- Response Format (Medium)
- Documentation (Medium)

---

### git-workflow

Git best practices, commit conventions, branching strategies, and PR workflows. Guidelines for maintaining clean, useful git history.

**Use when:**
- Writing commit messages
- Creating branches
- Setting up git workflows
- Reviewing pull requests

**Categories covered:**
- Commit Messages (Critical)
- Branching Strategy (High)
- Pull Requests (High)
- History Management (Medium)
- Collaboration (Medium)

---

### testing-best-practices

Unit testing, integration testing, and TDD principles. Guidelines for writing effective, maintainable test suites.

**Use when:**
- Writing unit tests
- Writing integration tests
- Reviewing test code
- Improving test coverage

**Categories covered:**
- Test Structure (Critical)
- Test Isolation (Critical)
- Assertions (High)
- Test Data (High)
- Mocking (Medium)
- Coverage (Medium)
- Performance (Low)

---

## Installation

### Personal Skills (Recommended)

Copy to `~/.claude/skills/` for use across all projects:

```bash
git clone https://github.com/AsyrafHussin/agent-skills.git
cp -r agent-skills/skills/* ~/.claude/skills/
```

### Project Skills

Copy to `.claude/skills/` in specific projects:

```bash
cp -r agent-skills/skills/* /path/to/project/.claude/skills/
```

### Symlink (Development)

```bash
ln -s /path/to/agent-skills/skills/* ~/.claude/skills/
```

## Usage

Skills are automatically available once installed. Claude will use them when relevant tasks are detected.

**Examples:**
```
Review this React component for performance issues
```
```
Help me set up Zustand with React Query
```
```
Create a Laravel controller with proper validation
```
```
Check this form for accessibility issues
```

## Skill Structure

Each skill contains:
- `SKILL.md` - Instructions for the agent (required)
- `README.md` - Human-readable documentation
- `rules/` - Detailed rules with examples (progressive disclosure)

## License

MIT
