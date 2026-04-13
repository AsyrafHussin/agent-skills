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
