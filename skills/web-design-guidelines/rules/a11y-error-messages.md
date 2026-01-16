---
title: Provide Accessible Error Messages
impact: CRITICAL
impactDescription: WCAG 2.1 Level A - Error identification and suggestions
tags: accessibility, forms, errors, aria, validation
---

## Provide Accessible Error Messages

**Impact: CRITICAL (WCAG 2.1 Level A - Error identification and suggestions)**

Error messages must be perceivable, understandable, and programmatically associated with their related form fields. Users should be able to identify and correct errors easily.

## Bad Example

```html
<!-- Anti-pattern: Poor error handling -->
<form>
  <!-- Error only shown visually with color -->
  <div class="form-group error">
    <label for="email">Email</label>
    <input type="email" id="email" style="border-color: red;">
  </div>

  <!-- Error message not associated with input -->
  <div class="form-group">
    <label for="password">Password</label>
    <input type="password" id="password">
  </div>
  <div class="error-text">Password must be 8 characters</div>

  <!-- Generic error message -->
  <div class="error-message">
    Please correct the errors above.
  </div>

  <!-- Error not announced to screen readers -->
  <div class="form-group">
    <label for="phone">Phone</label>
    <input type="tel" id="phone">
    <span class="error">Invalid phone number</span>
  </div>
</form>

<script>
// Anti-pattern: Alert boxes for errors
function validateForm() {
  if (!isValid) {
    alert('Please fix the errors in the form');
  }
}
</script>
```

## Good Example

```html
<!-- Correct approach: Accessible error handling -->
<form novalidate aria-describedby="form-error-summary">
  <!-- Error summary at top of form -->
  <div id="form-error-summary"
       class="error-summary"
       role="alert"
       aria-live="polite"
       hidden>
    <h2>There were 2 errors with your submission</h2>
    <ul>
      <li><a href="#email">Email address is required</a></li>
      <li><a href="#password">Password must be at least 8 characters</a></li>
    </ul>
  </div>

  <!-- Field with error - complete implementation -->
  <div class="form-group">
    <label for="email">
      Email Address
      <span aria-hidden="true">*</span>
    </label>
    <input type="email"
           id="email"
           name="email"
           required
           aria-required="true"
           aria-invalid="true"
           aria-describedby="email-error email-hint">
    <p id="email-hint" class="hint-text">
      We'll never share your email with anyone
    </p>
    <p id="email-error" class="error-message" role="alert">
      <svg aria-hidden="true" class="error-icon"><!-- error icon --></svg>
      <span>Please enter a valid email address (e.g., name@example.com)</span>
    </p>
  </div>

  <!-- Field with error - visual and programmatic indicators -->
  <div class="form-group has-error">
    <label for="password">
      Password
      <span aria-hidden="true">*</span>
    </label>
    <input type="password"
           id="password"
           name="password"
           required
           aria-required="true"
           aria-invalid="true"
           aria-describedby="password-error password-requirements">
    <p id="password-requirements" class="hint-text">
      Must contain at least 8 characters, one uppercase, one number
    </p>
    <p id="password-error" class="error-message" role="alert">
      <svg aria-hidden="true" class="error-icon"><!-- error icon --></svg>
      <span>Password is too short. Please use at least 8 characters.</span>
    </p>
  </div>

  <!-- Success state after correction -->
  <div class="form-group has-success">
    <label for="username">Username</label>
    <input type="text"
           id="username"
           name="username"
           aria-invalid="false"
           aria-describedby="username-success">
    <p id="username-success" class="success-message">
      <svg aria-hidden="true" class="success-icon"><!-- checkmark --></svg>
      <span>Username is available</span>
    </p>
  </div>
</form>

<script>
function showError(input, message) {
  const formGroup = input.closest('.form-group');
  const errorElement = formGroup.querySelector('.error-message');

  // Set ARIA attributes
  input.setAttribute('aria-invalid', 'true');

  // Show error message
  errorElement.textContent = message;
  errorElement.hidden = false;

  // Move focus to input (for form submission errors)
  input.focus();
}

function clearError(input) {
  const formGroup = input.closest('.form-group');
  const errorElement = formGroup.querySelector('.error-message');

  input.setAttribute('aria-invalid', 'false');
  errorElement.hidden = true;
}

function showErrorSummary(errors) {
  const summary = document.getElementById('form-error-summary');
  const list = summary.querySelector('ul');

  // Build error list
  list.innerHTML = errors.map(error =>
    `<li><a href="#${error.fieldId}">${error.message}</a></li>`
  ).join('');

  // Update heading
  summary.querySelector('h2').textContent =
    `There ${errors.length === 1 ? 'was 1 error' : `were ${errors.length} errors`} with your submission`;

  // Show and focus summary
  summary.hidden = false;
  summary.focus();
}
</script>

<style>
.error-summary {
  background-color: #fef2f2;
  border: 2px solid #dc2626;
  border-radius: 4px;
  padding: 1rem;
  margin-bottom: 1.5rem;
}

.error-summary:focus {
  outline: 3px solid #005fcc;
  outline-offset: 2px;
}

.error-message {
  color: #dc2626;
  font-size: 0.875rem;
  margin-top: 0.5rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.has-error input {
  border-color: #dc2626;
  border-width: 2px;
}

.has-error input:focus {
  outline-color: #dc2626;
  box-shadow: 0 0 0 3px rgba(220, 38, 38, 0.2);
}

.success-message {
  color: #059669;
  font-size: 0.875rem;
  margin-top: 0.5rem;
}
</style>
```

## Why

Accessible error messages are crucial because:

1. **Screen Reader Announcement**: Errors must be announced to screen reader users through ARIA live regions or focus management.

2. **Color Independence**: Color alone cannot indicate errors; icons and text are needed.

3. **Error Identification**: Users must understand which field has an error and what's wrong.

4. **Error Recovery**: Clear instructions help users fix mistakes.

5. **WCAG Compliance**: Multiple success criteria address error handling.

Error message best practices:

1. **Use `aria-invalid`**: Mark erroneous inputs programmatically
2. **Associate with Input**: Use `aria-describedby` to link error text
3. **Use Live Regions**: `role="alert"` or `aria-live` for dynamic errors
4. **Error Summary**: List all errors at form top with links to fields
5. **Move Focus**: Focus error summary or first error field on submission
6. **Be Specific**: "Email format is invalid" not "Invalid input"
7. **Provide Solutions**: Tell users how to fix the error
8. **Visual + Programmatic**: Combine icons, borders, and text
9. **Persist Until Fixed**: Keep errors visible until corrected
10. **Test with Screen Readers**: Verify errors are announced properly
