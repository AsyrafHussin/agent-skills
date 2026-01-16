---
title: Use Placeholders Appropriately
impact: MEDIUM
impactDescription: Prevents accessibility and usability issues
tags: forms, placeholders, labels, accessibility
---

## Use Placeholders Appropriately

**Impact: MEDIUM (Prevents accessibility and usability issues)**

Use placeholders appropriately as supplementary hints, never as replacements for labels. Placeholders have significant accessibility and usability limitations.

## Bad Example

```html
<!-- Anti-pattern: Placeholders as labels -->
<form>
  <!-- No visible labels - only placeholders -->
  <input type="text" placeholder="First Name">
  <input type="text" placeholder="Last Name">
  <input type="email" placeholder="Email Address">
  <input type="password" placeholder="Password (min 8 characters)">

  <!-- Required indicator in placeholder -->
  <input type="text" placeholder="Phone Number *">

  <!-- Critical information in placeholder -->
  <input type="text" placeholder="Enter date as MM/DD/YYYY">

  <!-- Placeholder that looks like entered text -->
  <input type="text"
         placeholder="John Smith"
         style="color: #333;">

  <!-- Long placeholder that gets truncated -->
  <input type="text"
         placeholder="Enter your full legal name as it appears on your government ID">
</form>

<style>
/* Anti-pattern: Hiding labels */
label { display: none; }
</style>
```

## Good Example

```html
<!-- Correct approach: Labels with optional placeholder hints -->
<form>
  <div class="form-group">
    <label for="first-name">First Name</label>
    <input type="text"
           id="first-name"
           name="firstName"
           autocomplete="given-name">
    <!-- No placeholder needed - label is clear -->
  </div>

  <div class="form-group">
    <label for="email">
      Email Address
      <span class="required" aria-hidden="true">*</span>
    </label>
    <input type="email"
           id="email"
           name="email"
           required
           aria-required="true"
           placeholder="name@example.com"
           autocomplete="email">
    <!-- Placeholder shows format example -->
  </div>

  <div class="form-group">
    <label for="phone">Phone Number</label>
    <input type="tel"
           id="phone"
           name="phone"
           placeholder="(555) 123-4567"
           aria-describedby="phone-hint"
           autocomplete="tel">
    <p id="phone-hint" class="hint">Include area code</p>
    <!-- Hint text for persistent instructions -->
  </div>

  <div class="form-group">
    <label for="date">Event Date</label>
    <input type="text"
           id="date"
           name="date"
           aria-describedby="date-format"
           placeholder="MM/DD/YYYY">
    <p id="date-format" class="hint">Format: MM/DD/YYYY</p>
    <!-- Important format info in persistent hint -->
  </div>

  <div class="form-group">
    <label for="password">
      Password
      <span class="required" aria-hidden="true">*</span>
    </label>
    <input type="password"
           id="password"
           name="password"
           required
           aria-required="true"
           aria-describedby="password-requirements"
           minlength="8"
           autocomplete="new-password">
    <p id="password-requirements" class="hint">
      Must be at least 8 characters with one uppercase letter and one number
    </p>
    <!-- Requirements in persistent text, not placeholder -->
  </div>

  <div class="form-group">
    <label for="search">Search Products</label>
    <input type="search"
           id="search"
           name="search"
           placeholder="e.g., blue widget, SKU-1234"
           autocomplete="off">
    <!-- Placeholder shows example queries -->
  </div>

  <div class="form-group">
    <label for="bio">Bio</label>
    <textarea id="bio"
              name="bio"
              placeholder="Tell us about yourself..."
              aria-describedby="bio-hint"
              rows="4"></textarea>
    <p id="bio-hint" class="hint">Optional. Max 500 characters.</p>
  </div>
</form>

<style>
/* Placeholder styling - clearly different from entered text */
::placeholder {
  color: #9ca3af;
  opacity: 1; /* Firefox fix */
  font-style: italic;
}

/* Hint text is always visible */
.hint {
  color: #6b7280;
  font-size: 0.875rem;
  margin-top: 0.25rem;
}

/* Never hide labels */
label {
  display: block;
  font-weight: 500;
  margin-bottom: 0.25rem;
}
</style>
```

## Why

Placeholder-only patterns have serious problems:

1. **Disappearing Labels**: When users click into a field, the placeholder vanishes, leaving no context for what data is expected.

2. **Memory Burden**: Users must remember what each field requires while typing.

3. **Low Contrast**: Default placeholder text often has insufficient contrast (accessibility violation).

4. **Translation Issues**: Placeholders may not translate well or may truncate in other languages.

5. **Screen Reader Support**: Some screen readers don't announce placeholders, or announce them only once.

6. **Validation Confusion**: Users may think the placeholder text is pre-filled content.

7. **Autofill Issues**: Browser autofill may not correctly identify fields without labels.

8. **Cognitive Load**: Users with cognitive disabilities may struggle to remember disappeared instructions.

When placeholders are appropriate:

- **Format examples**: `(555) 123-4567` for phone
- **Short hints**: `e.g., blue widget` for search
- **Optional context**: `@username` for social handles

Placeholder best practices:

1. Always use visible labels
2. Keep placeholders short and supplementary
3. Don't put critical information in placeholders
4. Don't put required field indicators in placeholders
5. Use `aria-describedby` for persistent hints
6. Style placeholders distinctly from entered text
7. Ensure placeholder contrast meets WCAG guidelines
8. Test with the placeholder hidden to verify usability
