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
