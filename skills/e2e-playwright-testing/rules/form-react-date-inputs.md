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
