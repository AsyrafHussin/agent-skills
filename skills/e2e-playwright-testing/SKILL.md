---
name: e2e-playwright-testing
description: End-to-end testing with Playwright for web applications. Use when writing E2E tests, browser automation, form submission testing, or user flow testing. Triggers on "playwright", "e2e test", "browser test", "end-to-end", "form flow testing", or test files in tests/e2e/.
license: MIT
metadata:
  author: Agent Skills Contributors
  version: "1.0.0"
  playwrightVersion: "1.50+"
  nodeVersion: "18+"
---

# E2E Playwright Testing

> Patterns and conventions for reliable end-to-end browser testing with Playwright.

Comprehensive E2E testing guide for web applications. Contains 8 rules across 6 categories covering locator strategies, authentication reuse, form testing (including React/SPA-specific gotchas), assertions, test organization, reliability, and CI/CD configuration.

## Stack Detection

**Before writing or reviewing E2E tests, detect the project stack:**

### Step 1 — Check for Playwright

```bash
# Look in package.json devDependencies:
# "@playwright/test" → Playwright is installed
# Check for playwright.config.js or playwright.config.ts
```

- If `@playwright/test` is present → **use Playwright patterns from this skill**
- If `cypress` is present → this skill does not apply

### Step 2 — Detect Frontend Framework

```bash
# Check package.json dependencies:
# "react" + "@inertiajs/react" → React + Inertia.js (SPA with server routing)
# "react" + "react-router" → React SPA
# "vue" → Vue.js
# "next" → Next.js
```

**Why this matters:**
- **React + Inertia.js**: Use `waitForURL` not `waitForLoadState('networkidle')` — Inertia uses `history.pushState`
- **React controlled inputs**: `fill()` works for text but `keyboard.type()` needed for date/time
- **SPA navigation**: Page doesn't do full reload — assertions must wait for content, not network idle

### Step 3 — Detect Auth Pattern

```bash
# Check for session-based (Laravel Sanctum) or token-based (JWT) auth
# Session: storageState saves cookies
# Token: may need to save tokens to localStorage
```

### Step 4 — Detect Rate Limiting

```bash
# Check routes for throttle middleware
# If present in dev, tests will hit 429 after ~5 login attempts
# Solution: production-only throttle or increase limits in test env
```

---

## Metadata

- **Version:** 1.0.0
- **Rule Count:** 8 rules across 6 categories
- **License:** MIT

## When to Apply

- Writing or reviewing Playwright E2E tests
- Setting up E2E testing for a new project
- Debugging flaky browser tests
- Testing form submissions, authentication flows, or user interactions
- Choosing locator strategies for elements
- Configuring Playwright for CI/CD

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Locators | CRITICAL | `loc` |
| 2 | Authentication | CRITICAL | `auth` |
| 3 | Assertions | HIGH | `assert` |
| 4 | Forms & Inputs | HIGH | `form` |
| 5 | Test Organization | MEDIUM | `org` |
| 6 | Reliability | MEDIUM | `rel` |

## Quick Reference

### 1. Locators (CRITICAL)

- `loc-prefer-role-locators` - Use getByRole/getByLabel over CSS selectors
- `loc-strict-mode` - Handle strict mode violations with exact/first/scoped

### 2. Authentication (CRITICAL)

- `auth-storage-state` - Reuse login state via setup project pattern

### 3. Assertions (HIGH)

- `assert-web-first` - Use auto-retrying expect(locator) assertions

### 4. Forms & Inputs (HIGH)

- `form-react-date-inputs` - Use keyboard.type() for date/time in React apps
- `form-custom-checkboxes` - Handle sr-only checkbox components

### 5. Test Organization (MEDIUM)

- `org-mirror-routes` - Directory structure mirrors route groups

### 6. Reliability (MEDIUM)

- `rel-no-wait-for-timeout` - Never use arbitrary waitForTimeout

## Essential Patterns

### Locator Priority

```javascript
// 1st: Role (best — mirrors accessibility)
page.getByRole('button', { name: 'Submit' })
page.getByRole('tab', { name: 'Network' })
page.getByRole('heading', { name: 'Dashboard' })

// 2nd: Label (for form fields)
page.getByLabel('Email')

// 3rd: Text (for static content, use exact when ambiguous)
page.getByText('Welcome', { exact: true })

// 4th: ID (for inputs without proper labels)
page.locator('#password')

// Last resort: CSS selector
page.locator('button.submit-btn')
```

### Auth Setup Pattern

```javascript
// Setup project logs in once per role, all tests reuse the state
// 3 logins for 90+ tests (not 90 logins)

// In test files:
test.use({ role: 'customer' });
test('shows dashboard', async ({ authedPage: page }) => {
    await page.goto('/dashboard');
    await expect(page.getByRole('heading', { level: 1 })).toBeVisible();
});
```

### React Date Input Gotcha

```javascript
// fill() doesn't trigger React onChange for date/time inputs
// Use keyboard.type() instead:
await dateInput.click();
await page.keyboard.type('16042026');  // DDMMYYYY

await timeInput.click();
await page.keyboard.type('1000AM');    // 10:00 AM
```

### Strict Mode Fix

```javascript
// Bad: matches "Verified" AND "Unverified"
page.getByRole('button', { name: 'Verified' })

// Good: exact match
page.getByRole('button', { name: 'Verified', exact: true })

// Bad: matches heading AND breadcrumb
page.getByText('POS Demo')

// Good: use role to disambiguate
page.getByRole('heading', { name: 'POS Demo' })
```

### Parallelism Decision

```javascript
// Default Playwright: fullyParallel: true
// Use this when: tests are independent, each test creates its own data

// Override to sequential: workers: 1
// Use this when: tests share a seeded database with mutable state
// Without this, one test's buy transaction changes the balance another test expects
```

## Configuration Template

```javascript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
    testDir: './specs',
    fullyParallel: false,              // true if tests are independent
    workers: 1,                        // increase if no shared mutable state
    forbidOnly: !!process.env.CI,
    retries: process.env.CI ? 2 : 0,
    reporter: [['html', { open: 'never' }], ['list']],
    use: {
        baseURL: process.env.PLAYWRIGHT_BASE_URL || 'http://localhost:8000',
        trace: 'on-first-retry',
        screenshot: 'only-on-failure',
        video: 'retain-on-failure',
    },
    projects: [
        { name: 'auth-setup', testMatch: /auth\.setup\.js$/, testDir: './auth' },
        { name: 'chrome', use: { ...devices['Desktop Chrome'] }, dependencies: ['auth-setup'] },
        { name: 'mobile', use: { ...devices['iPhone 14'] }, dependencies: ['auth-setup'], testMatch: /responsive/ },
    ],
});
```

## References

- [Playwright Documentation](https://playwright.dev)
- [Best Practices](https://playwright.dev/docs/best-practices)
- [Locators Guide](https://playwright.dev/docs/locators)
- [Authentication Guide](https://playwright.dev/docs/auth)
- [Test Assertions](https://playwright.dev/docs/test-assertions)
- [Auto-waiting](https://playwright.dev/docs/actionability)

## Full Compiled Document

For the complete guide with all rules expanded: `AGENTS.md`
