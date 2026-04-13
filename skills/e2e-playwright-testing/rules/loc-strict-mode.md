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
