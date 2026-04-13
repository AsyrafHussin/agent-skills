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


---

# Full Rule Reference


---

---
title: Prefer Role-Based Locators
impact: CRITICAL
impactDescription: "test resilience to UI changes"
tags: locators, accessibility, getByRole, best-practice
---

## Prefer Role-Based Locators

**Impact: CRITICAL (test resilience to UI changes)**

Use `getByRole`, `getByLabel`, and `getByText` over CSS selectors or test IDs. Role-based locators mirror how users and assistive technology interact with the page, making tests resilient to refactoring.

## Incorrect

```javascript
// Bad: CSS selectors break when classes change
await page.locator('.btn-primary').click();
await page.locator('#submit-form').click();
await page.locator('div > form > button:nth-child(2)').click();

// Bad: data-testid couples tests to implementation
await page.locator('[data-testid="login-button"]').click();
```

**Problems:**
- CSS classes are styling concerns, not behavior contracts
- Selectors break on every UI redesign
- Tests don't verify accessibility (screen readers can't find `.btn-primary`)
- `data-testid` adds noise to production HTML

## Correct

```javascript
// Good: Role-based — mirrors user interaction
await page.getByRole('button', { name: 'Sign In' }).click();
await page.getByRole('tab', { name: 'Network' }).click();
await page.getByRole('heading', { name: 'Dashboard' }).toBeVisible();

// Good: Label-based — matches form field labels
await page.getByLabel('Email').fill('user@example.com');

// Good: Text-based — for non-interactive content
await expect(page.getByText('Welcome back')).toBeVisible();

// Good: Use exact when text is ambiguous
await page.getByRole('button', { name: 'Verified', exact: true });
```

**Priority order:**
1. `getByRole` — buttons, tabs, headings, links, checkboxes
2. `getByLabel` — form inputs with labels
3. `getByText` — static content, with `{ exact: true }` when needed
4. `locator('#id')` — for inputs without labels (e.g., `#password`)
5. CSS selectors — last resort for complex DOM structures

Reference: [Playwright Docs — Locators](https://playwright.dev/docs/locators)

---

---
title: Handle Strict Mode Violations
impact: CRITICAL
impactDescription: "prevents test failures from ambiguous selectors"
tags: locators, strict-mode, exact, disambiguation
---

## Handle Strict Mode Violations

**Impact: CRITICAL (prevents test failures from ambiguous selectors)**

Playwright runs in strict mode by default — locators that match multiple elements throw an error. Handle this proactively with `{ exact: true }`, `.first()`, or scoped locators.

## Incorrect

```javascript
// Bad: "Verified" also matches "Unverified"
await page.getByRole('button', { name: 'Verified' }).click();

// Bad: "POS Demo" appears in heading AND breadcrumb
await expect(page.getByText('POS Demo')).toBeVisible();

// Bad: "Item Type" appears in form AND summary
await expect(page.getByText('Item Type')).toBeVisible();

// Bad: "Convert" matches tab name AND submit button
await page.getByRole('button', { name: /convert/i }).click();
```

**Problems:**
- Strict mode throws: "resolved to 2 elements"
- Tests fail with confusing error messages
- Different elements have the same visible text

## Correct

```javascript
// Good: exact match prevents substring matching
await page.getByRole('button', { name: 'Verified', exact: true }).click();

// Good: use role to disambiguate heading vs breadcrumb
await expect(page.getByRole('heading', { name: 'POS Demo' })).toBeVisible();

// Good: scope to a specific container
await expect(page.locator('.form-section').getByText('Item Type')).toBeVisible();

// Good: use specific button text
await page.getByRole('button', { name: /convert 10 sp/i }).click();

// Good: use .first() when you know the first match is correct
await page.getByText('Dashboard').first().click();

// Good: filter with parent context
await page.locator('button').filter({ hasText: 'Arif Azmi' }).first().click();
```

**Strategies (in order of preference):**
1. `{ exact: true }` — prevents "Verified" matching "Unverified"
2. `getByRole('heading', ...)` — disambiguates heading from breadcrumb/sidebar
3. Scoped locator — `container.getByText(...)` limits search area
4. `.first()` — when the first match is always correct
5. `.filter({ hasText: ... })` — combine with other locators

Reference: [Playwright Docs — Strict Mode](https://playwright.dev/docs/locators#strictness)

---

---
title: Reuse Authentication with Storage State
impact: CRITICAL
impactDescription: "eliminates login overhead and rate-limit failures"
tags: authentication, storage-state, setup-project, session
---

## Reuse Authentication with Storage State

**Impact: CRITICAL (eliminates login overhead and rate-limit failures)**

Log in once per role in a setup project, save browser state to JSON files, and reuse via `storageState` in all tests. Never log in per-test.

## Incorrect

```javascript
// Bad: Logging in at the start of every test
test('dashboard shows data', async ({ page }) => {
    await page.goto('/login');
    await page.getByLabel('Email').fill('user@example.com');
    await page.locator('#password').fill('password');
    await page.getByRole('button', { name: 'Sign In' }).click();
    await page.waitForURL('**/dashboard');

    // Now the actual test...
    await expect(page.getByText('Welcome')).toBeVisible();
});

test('profile shows name', async ({ page }) => {
    // Same login repeated — slow, hits rate limits
    await page.goto('/login');
    await page.getByLabel('Email').fill('user@example.com');
    // ...
});
```

**Problems:**
- Each test wastes 1-2 seconds on login
- Login rate limits (429) block the entire suite after ~5 tests
- Login failures cascade — every test fails, not just auth tests

## Correct

```javascript
// auth/auth.setup.js — runs once before all tests
import { test as setup, expect } from '@playwright/test';
import path from 'path';

const AUTH_DIR = path.join(import.meta.dirname, '..', '.auth');

const users = [
    { name: 'customer', file: 'customer.json', email: 'customer@example.com' },
    { name: 'admin', file: 'admin.json', email: 'admin@example.com' },
];

for (const user of users) {
    setup(`authenticate as ${user.name}`, async ({ page }) => {
        await page.goto('/login');
        await page.getByLabel('Email').fill(user.email);
        await page.locator('#password').fill('password');
        await page.getByRole('button', { name: 'Sign In' }).click();
        await page.waitForURL('**/dashboard');

        await page.context().storageState({ path: path.join(AUTH_DIR, user.file) });
    });
}
```

```javascript
// fixtures/test.js — custom fixture for role-based auth
import { test as base } from '@playwright/test';
import path from 'path';

const AUTH_DIR = path.join(import.meta.dirname, '..', '.auth');

export const test = base.extend({
    role: ['customer', { option: true }],
    authedPage: async ({ browser, role }, use) => {
        const context = await browser.newContext({
            storageState: path.join(AUTH_DIR, `${role}.json`),
        });
        const page = await context.newPage();
        await use(page);
        await context.close();
    },
});
```

```javascript
// specs/dashboard.spec.js — tests get pre-authenticated pages
import { test, expect } from '../fixtures/test.js';

test.describe('Dashboard', () => {
    test.use({ role: 'customer' });

    test('shows welcome message', async ({ authedPage: page }) => {
        await page.goto('/dashboard');
        await expect(page.getByText('Welcome')).toBeVisible();
    });
});
```

```javascript
// playwright.config.js — setup project runs first
export default defineConfig({
    projects: [
        { name: 'auth-setup', testMatch: /auth\.setup\.js$/, testDir: './auth' },
        { name: 'chrome', use: { ...devices['Desktop Chrome'] }, dependencies: ['auth-setup'] },
    ],
});
```

**Benefits:**
- Login happens once per role (3 logins for 90+ tests)
- No rate-limit issues even with strict throttling
- Each test starts instantly with pre-loaded cookies
- Role switching is a simple `test.use({ role: 'admin' })`

Reference: [Playwright Docs — Authentication](https://playwright.dev/docs/auth)

---

---
title: Use Web-First Assertions
impact: HIGH
impactDescription: "eliminates flaky timing-dependent tests"
tags: assertions, auto-retry, web-first, expect
---

## Use Web-First Assertions

**Impact: HIGH (eliminates flaky timing-dependent tests)**

Playwright's `expect(locator)` assertions auto-retry until the condition is met or timeout. Never use manual waits or DOM queries followed by manual checks.

## Incorrect

```javascript
// Bad: Manual DOM query — no auto-retry, race condition
const button = await page.$('.submit-btn');
expect(button).not.toBeNull();

// Bad: Using waitForTimeout — arbitrary delay, still flaky
await page.waitForTimeout(2000);
const text = await page.textContent('.status');
expect(text).toBe('Success');

// Bad: isVisible() returns immediately — no retry
const visible = await page.locator('.toast').isVisible();
expect(visible).toBe(true);

// Bad: Checking count for existence
await expect(page.locator('.item')).toHaveCount(1);
```

**Problems:**
- `page.$()` returns null if element hasn't rendered yet
- `waitForTimeout` wastes time on fast environments, still flaky on slow ones
- `isVisible()` is a snapshot — doesn't wait for animations or lazy rendering
- `toHaveCount(1)` is fragile — `toBeVisible()` expresses intent better

## Correct

```javascript
// Good: Web-first assertions auto-retry with configurable timeout
await expect(page.getByText('Success')).toBeVisible();
await expect(page.getByRole('button')).toBeEnabled();
await expect(page.getByRole('button')).toBeDisabled();
await expect(page.getByRole('dialog')).toBeVisible();
await expect(page.getByRole('dialog')).not.toBeVisible();

// Good: Custom timeout for slow operations
await expect(page.getByText('Uploaded')).toBeVisible({ timeout: 10_000 });

// Good: Text content assertions
await expect(page.getByRole('heading')).toHaveText('Dashboard');
await expect(page.locator('#balance')).toContainText('RM');

// Good: Attribute assertions
await expect(page.locator('input')).toHaveValue('100');
await expect(page.locator('html')).toHaveClass(/dark/);

// Good: URL assertion after navigation
await expect(page).toHaveURL(/\/dashboard/);
```

**Common assertions:**
- `toBeVisible()` / `toBeHidden()` — element visibility
- `toBeEnabled()` / `toBeDisabled()` — button/input state
- `toHaveText()` / `toContainText()` — text content
- `toHaveValue()` — input value
- `toHaveAttribute()` — HTML attribute
- `toHaveURL()` — page URL
- `toHaveClass()` — CSS class

Reference: [Playwright Docs — Assertions](https://playwright.dev/docs/test-assertions)

---

---
title: Handle Custom Checkbox Components
impact: HIGH
impactDescription: "prevents click-no-effect bugs on styled checkboxes"
tags: forms, checkbox, sr-only, custom-components, react
---

## Handle Custom Checkbox Components

**Impact: HIGH (prevents click-no-effect bugs on styled checkboxes)**

Custom checkbox components (like those using Tailwind's `sr-only` pattern) hide the native `<input>` and render a styled `<div>`. Playwright's `.check()` may not trigger React's `onChange` on hidden inputs. Multiple `<label htmlFor>` elements pointing to the same input cause double-toggle (check then uncheck).

## Incorrect

```javascript
// Bad: .check() on sr-only input — may not trigger React onChange
await page.locator('#terms').check();

// Bad: .check({ force: true }) — bypasses visibility but still no React event
await page.locator('#terms').check({ force: true });

// Bad: clicking a label when there are TWO labels for the same input
// <Checkbox id="terms" />          ← has its own <label> wrapper
// <label htmlFor="terms">I agree</label>  ← second label
// Clicking either label toggles twice = no change
await page.locator('label[for="terms"]').click();
```

**Problems:**
- `sr-only` inputs are invisible — `.check()` can't find them
- `force: true` clicks the element but React doesn't see the event
- Two `<label htmlFor>` elements cause double-toggle (browser behavior)
- Test shows checkbox as "checked" visually but React state is `false`

## Correct

```javascript
// Good: Click the component's own <label> wrapper (contains the visual checkbox)
await page.locator('label:has(#terms)').click();

// Good: Click the T&C text that toggles via onClick handler
await page.getByText('I agree to the terms').click();

// Good: For components with only one label, click the label directly
await page.locator('label[for="remember"]').click();

// Good: Verify the checkbox state with web-first assertion
await expect(page.locator('#terms')).toBeChecked();
```

**Fix the component if two labels exist (code bug):**
```jsx
// Bad: Two labels for same input — browser toggles twice
<Checkbox id="terms" />
<label htmlFor="terms">I agree</label>

// Good: Use <span> for the text, let Checkbox own the label
<Checkbox id="terms" />
<span onClick={() => setAgreed(v => !v)}>I agree</span>
```

Reference: [MDN — Label Element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/label)

---

---
title: Handle React Controlled Date and Time Inputs
impact: HIGH
impactDescription: "prevents silent form state desync"
tags: forms, date-input, time-input, react, controlled-components
---

## Handle React Controlled Date and Time Inputs

**Impact: HIGH (prevents silent form state desync)**

Playwright's `fill()` sets the DOM value but does not always trigger React's `onChange` for date and time inputs. The HTML value updates but React state stays empty, causing form validation to fail silently.

## Incorrect

```javascript
// Bad: fill() updates DOM but React state stays empty
await page.locator('#date').fill('2026-04-14');
await page.locator('#time').fill('10:00');

// The submit button remains disabled because React state is:
// { preferred_date: '', preferred_time: '' }
// Even though the inputs visually show the values
```

**Problems:**
- `fill()` dispatches events that React date/time inputs may not listen to
- Form appears filled visually but React state is empty
- Submit button stays disabled, test times out with confusing failure
- Only affects date/time input types — text inputs work fine with `fill()`

## Correct

```javascript
// Good: Use keyboard.type() to simulate real user input
// Date inputs accept digits in the browser's locale format (DD/MM/YYYY or MM/DD/YYYY)
const dateInput = page.locator('#preferred-date');
await dateInput.scrollIntoViewIfNeeded();
await dateInput.click();
await page.keyboard.type('16042026'); // DDMMYYYY for DD/MM/YYYY locale

// Time inputs accept digits + AM/PM
const timeInput = page.locator('#preferred-time');
await timeInput.click();
await page.keyboard.type('1000AM'); // 10:00 AM

// Brief wait needed: keyboard.type() on date inputs triggers React state
// updates asynchronously — 200ms allows the controlled component to sync
await page.waitForTimeout(200);
```

```javascript
// Alternative: Use evaluate to set value + dispatch native events
await page.evaluate((val) => {
    const el = document.getElementById('preferred-date');
    const setter = Object.getOwnPropertyDescriptor(
        window.HTMLInputElement.prototype, 'value'
    ).set;
    setter.call(el, val);
    el.dispatchEvent(new Event('input', { bubbles: true }));
    el.dispatchEvent(new Event('change', { bubbles: true }));
}, '2026-04-16');
```

```javascript
// For standard text/number inputs, fill() works correctly
await page.locator('#amount').fill('100');     // Works
await page.locator('#email').fill('a@b.com');  // Works
await page.locator('#weight').fill('25.5');    // Works
```

**When to use `keyboard.type()` vs `fill()`:**
- `fill()` — text, number, email, password, search inputs
- `keyboard.type()` — date, time, datetime-local inputs in React apps
- Both require `.click()` first to focus the input

Reference: [Playwright Docs — Input](https://playwright.dev/docs/input)

---

---
title: Mirror Route Structure in Test Organization
impact: MEDIUM
impactDescription: "scalable test organization as app grows"
tags: organization, structure, routes, feature-based
---

## Mirror Route Structure in Test Organization

**Impact: MEDIUM (scalable test organization as app grows)**

Organize E2E test files to mirror your application's route groups. This makes tests discoverable and scalable as features are added.

## Incorrect

```
tests/e2e/specs/
  test1.spec.js          # What does this test?
  test2.spec.js          # No grouping
  all-tests.spec.js      # One giant file with 200 tests
  login-and-buy.spec.js  # Mixed concerns
```

**Problems:**
- Can't find the test for a specific feature
- Adding a new feature means figuring out which file to edit
- One giant file causes merge conflicts
- Mixed concerns make failures hard to diagnose

## Correct

```
tests/e2e/specs/
  auth/
    login.spec.js
    register.spec.js
  customer/                    # matches role:customer routes
    egold/
      buy.spec.js
      sell.spec.js
      redeem.spec.js
      kyc-gate.spec.js
    gold-saving/
      new-agreement.spec.js
    special-trade-in/
      new-agreement.spec.js
    semantan-point/
      convert.spec.js
    team/
      member-profile.spec.js
  admin/                       # matches role:admin routes
    customers/
      kyc-approve.spec.js
    egold/
      approve-reject.spec.js
    pos-demo/
      transaction.spec.js
  management/                  # matches role:superadmin routes
    branches/
      create-edit.spec.js
    gold-price/
      add-price.spec.js
  ui/                          # cross-cutting concerns
    dark-mode.spec.js
    responsive.spec.js
    navigation.spec.js
    global-search.spec.js
```

**Naming convention:**
- Folders match route prefixes (`/admin/*` → `admin/`)
- Files match the feature or page name
- `ui/` folder for cross-cutting concerns (theme, responsive, navigation)
- One `describe` block per file, focused on one page/feature

Reference: [Playwright Docs — Test Organization](https://playwright.dev/docs/test-configuration)

---

---
title: Never Use Arbitrary waitForTimeout
impact: MEDIUM
impactDescription: "prevents slow and flaky tests"
tags: reliability, waiting, auto-wait, flaky-tests
---

## Never Use Arbitrary waitForTimeout

**Impact: MEDIUM (prevents slow and flaky tests)**

Playwright auto-waits for elements before interacting. Adding `waitForTimeout` is almost always a sign of a missing assertion or incorrect locator strategy.

## Incorrect

```javascript
// Bad: Waiting for toast to appear
await page.click('#submit');
await page.waitForTimeout(3000);
await expect(page.getByText('Success')).toBeVisible();

// Bad: Waiting for page transition
await page.click('a[href="/dashboard"]');
await page.waitForTimeout(2000);
await expect(page.getByRole('heading')).toBeVisible();

// Bad: Waiting for React re-render
await page.fill('#amount', '100');
await page.waitForTimeout(1000);
const isEnabled = await page.locator('#submit').isEnabled();
```

**Problems:**
- 3 second wait runs on every test even when page loads in 200ms
- On slow CI, 2 seconds may not be enough — still flaky
- Accumulates across test suite (30 tests x 3s = 90s wasted)

## Correct

```javascript
// Good: Wait for the element directly — auto-retries until visible
await page.click('#submit');
await expect(page.getByText('Success')).toBeVisible({ timeout: 8_000 });

// Good: Wait for URL change after navigation
await page.click('a[href="/dashboard"]');
await page.waitForURL('**/dashboard');
await expect(page.getByRole('heading')).toBeVisible();

// Good: Wait for button state to change
await page.fill('#amount', '100');
await expect(page.locator('#submit')).toBeEnabled({ timeout: 3_000 });

// Good: Wait for network request to complete
await Promise.all([
    page.waitForResponse('/api/data'),
    page.click('#load-data'),
]);
```

**Acceptable uses of waitForTimeout (rare):**
- After `keyboard.type()` on date inputs — React needs a tick to process (200ms max)
- After clipboard operations — OS-level delay
- Never more than 500ms, and always with a comment explaining why

Reference: [Playwright Docs — Auto-waiting](https://playwright.dev/docs/actionability)
