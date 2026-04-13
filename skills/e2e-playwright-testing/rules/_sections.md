# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. Locators (loc)

**Impact:** CRITICAL
**Description:** Choosing the right locators is the foundation of reliable E2E tests. Prefer user-facing attributes (role, label, text) over implementation details (CSS classes, IDs). Use `getByRole` as the primary strategy, `getByLabel`/`getByText` as secondary, and CSS/XPath only as a last resort. Handle strict mode violations with `{ exact: true }` or scoped locators.

## 2. Authentication (auth)

**Impact:** CRITICAL
**Description:** Authentication is the most common E2E bottleneck. Log in once per role using the setup project pattern, save browser state (cookies + localStorage) to JSON files, and reuse via `storageState` in test contexts. Never log in per-test — it wastes time and creates flaky rate-limit failures.

## 3. Assertions (assert)

**Impact:** HIGH
**Description:** Use Playwright's web-first assertions (`expect(locator)`) which auto-retry until the condition is met or timeout. Never use `page.$(selector)` + manual checks. Assert on user-visible state (text content, visibility, enabled/disabled), not internal implementation.

## 4. Forms & Inputs (form)

**Impact:** HIGH
**Description:** Form testing catches the most real bugs in web apps. Handle React controlled inputs correctly — `fill()` works for text but `keyboard.type()` is needed for date/time pickers. Custom checkbox components using `sr-only` patterns need special click handling to avoid double-toggle bugs.

## 5. Test Organization (org)

**Impact:** MEDIUM
**Description:** Organize tests by feature area mirroring your route structure. Use custom fixtures for role-based authentication. Keep test files focused — one `describe` block per page or feature.

## 6. Reliability (rel)

**Impact:** MEDIUM
**Description:** Flaky tests destroy CI confidence. Never use arbitrary `waitForTimeout` — use auto-waiting locators and web-first assertions instead. Run tests sequentially when they share mutable database state.
