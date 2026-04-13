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
