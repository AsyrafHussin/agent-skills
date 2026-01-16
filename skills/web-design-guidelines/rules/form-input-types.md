---
title: Use Correct Input Types
impact: HIGH
impactDescription: Improves mobile UX and reduces errors
tags: forms, input-types, mobile, ux
---

## Use Correct Input Types

**Impact: HIGH (Improves mobile UX and reduces errors)**

Use the correct HTML input types to provide appropriate virtual keyboards, validation, and user experience across devices.

## Bad Example

```html
<!-- Anti-pattern: Using generic text inputs for everything -->
<form>
  <label for="email">Email</label>
  <input type="text" id="email" name="email">

  <label for="phone">Phone</label>
  <input type="text" id="phone" name="phone">

  <label for="age">Age</label>
  <input type="text" id="age" name="age">

  <label for="website">Website</label>
  <input type="text" id="website" name="website">

  <label for="dob">Date of Birth</label>
  <input type="text" id="dob" name="dob" placeholder="MM/DD/YYYY">

  <label for="password">Password</label>
  <input type="text" id="password" name="password">

  <label for="color-choice">Favorite Color</label>
  <input type="text" id="color-choice" name="color">
</form>
```

## Good Example

```html
<!-- Correct approach: Semantic input types -->
<form>
  <!-- Email: brings up @ keyboard on mobile -->
  <label for="email">Email</label>
  <input type="email"
         id="email"
         name="email"
         autocomplete="email"
         inputmode="email"
         placeholder="name@example.com">

  <!-- Phone: numeric keypad on mobile -->
  <label for="phone">Phone</label>
  <input type="tel"
         id="phone"
         name="phone"
         autocomplete="tel"
         inputmode="tel"
         placeholder="+1 (555) 123-4567">

  <!-- Number: numeric input with spinners -->
  <label for="age">Age</label>
  <input type="number"
         id="age"
         name="age"
         min="0"
         max="150"
         inputmode="numeric">

  <!-- URL: optimized keyboard for URLs -->
  <label for="website">Website</label>
  <input type="url"
         id="website"
         name="website"
         autocomplete="url"
         inputmode="url"
         placeholder="https://example.com">

  <!-- Date: native date picker -->
  <label for="dob">Date of Birth</label>
  <input type="date"
         id="dob"
         name="dob"
         min="1900-01-01"
         max="2024-12-31"
         autocomplete="bday">

  <!-- Password: masked input with browser features -->
  <label for="password">Password</label>
  <input type="password"
         id="password"
         name="password"
         autocomplete="new-password"
         minlength="8">

  <!-- Color picker -->
  <label for="color-choice">Favorite Color</label>
  <input type="color"
         id="color-choice"
         name="color"
         value="#3b82f6">

  <!-- Search: may show clear button and search styling -->
  <label for="search">Search</label>
  <input type="search"
         id="search"
         name="search"
         autocomplete="off"
         inputmode="search">

  <!-- Range slider -->
  <label for="volume">Volume: <output id="volume-output">50</output>%</label>
  <input type="range"
         id="volume"
         name="volume"
         min="0"
         max="100"
         value="50"
         oninput="document.getElementById('volume-output').textContent = this.value">

  <!-- Time input -->
  <label for="meeting-time">Meeting Time</label>
  <input type="time"
         id="meeting-time"
         name="meeting-time"
         min="09:00"
         max="18:00">

  <!-- File upload -->
  <label for="document">Upload Document</label>
  <input type="file"
         id="document"
         name="document"
         accept=".pdf,.doc,.docx"
         multiple>

  <!-- Hidden field for form data -->
  <input type="hidden" name="csrf_token" value="abc123">
</form>
```

## Why

Correct input types provide significant benefits:

1. **Mobile Keyboards**: Different input types trigger appropriate virtual keyboards (numeric, email with @, URL with .com).

2. **Built-in Validation**: Browsers validate email, URL, number formats automatically.

3. **Native UI**: Date, time, color, and range inputs provide native pickers.

4. **Autofill**: Combined with autocomplete attributes, browsers can autofill forms accurately.

5. **Accessibility**: Screen readers announce the input type and constraints.

6. **Security**: Password fields mask input and may trigger password managers.

Available input types:

| Type | Use Case | Mobile Keyboard |
|------|----------|-----------------|
| `text` | Generic text | Standard |
| `email` | Email addresses | Email (@, .com) |
| `tel` | Phone numbers | Numeric |
| `url` | Web addresses | URL (/, .com) |
| `number` | Numeric values | Numeric |
| `password` | Sensitive data | Standard (masked) |
| `search` | Search queries | Search |
| `date` | Calendar dates | Date picker |
| `time` | Time values | Time picker |
| `datetime-local` | Date + time | DateTime picker |
| `month` | Month/year | Month picker |
| `week` | Week/year | Week picker |
| `color` | Color values | Color picker |
| `range` | Slider values | Slider |
| `file` | File uploads | File picker |
| `hidden` | Hidden data | None |

Best practices:
- Always pair with appropriate `autocomplete` attribute
- Use `inputmode` for fine-grained keyboard control
- Set `min`, `max`, `step` for numeric inputs
- Use `pattern` for custom validation regex
- Test on mobile devices to verify keyboard behavior
