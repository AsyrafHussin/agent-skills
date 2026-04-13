# E2E Playwright Testing

End-to-end testing patterns with Playwright for web applications.

## Overview

8 rules across 6 categories covering locator strategies, authentication reuse, form testing (including React-specific gotchas), assertions, test organization, and reliability.

## Key Patterns

- **Locators:** Role-based over CSS selectors, strict mode handling
- **Auth:** Setup project + storageState for login reuse
- **Forms:** keyboard.type() for React date/time inputs, sr-only checkbox handling
- **Reliability:** No arbitrary waits, single worker for shared DB, SPA navigation

## Based On

Patterns derived from real E2E implementation on a Laravel + React + Inertia.js gold trading platform (35+ pages, 94 Playwright tests). Bugs discovered during implementation informed the rules:

1. CSRF 419 failures from rate limiting during auth
2. React date inputs not triggering onChange with fill()
3. Double-label checkbox toggling twice (check + uncheck)
4. Strict mode violations from ambiguous text locators

## Installation

Copy the `e2e-playwright-testing` folder to your `.claude/skills/` directory.
