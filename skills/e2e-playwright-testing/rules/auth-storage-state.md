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
