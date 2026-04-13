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
