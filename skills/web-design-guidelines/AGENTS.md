# Web Design Guidelines - Complete Reference

**Version:** 1.0.0
**Organization:** Web Accessibility Initiative (WAI)
**Date:** January 2026
**License:** MIT

## Abstract

Comprehensive UI/UX and accessibility guidelines for building inclusive, performant web interfaces. Contains 21+ rules across 8 categories focused on accessibility, forms, animations, and performance. Each rule includes detailed explanations, real-world examples comparing incorrect vs. correct implementations, and specific WCAG compliance information to guide accessible web development and code review.

## References

- [WCAG 2.2 Guidelines](https://www.w3.org/WAI/WCAG22/)
- [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/)
- [ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)
- [Web.dev Accessibility](https://web.dev/accessibility/)
- [Learn Accessibility](https://web.dev/learn/accessibility/)
- [Prefers Reduced Motion](https://web.dev/articles/prefers-reduced-motion)
- [Cumulative Layout Shift](https://web.dev/articles/cls)
- [Optimize CLS](https://web.dev/articles/optimize-cls)
- [MDN Accessibility](https://developer.mozilla.org/en-US/docs/Web/Accessibility)
- [A11y Project](https://a11yproject.com/)

## Overview

This guide is organized into 8 categories, prioritized by WCAG compliance levels and real-world impact on users:

1. **Accessibility - Semantic Structure** (CRITICAL) - Foundation for assistive technology
2. **Accessibility - Keyboard & Focus** (CRITICAL) - Essential for keyboard-only users
3. **Accessibility - Visual & Color** (CRITICAL) - Perceivable content for all users
4. **Forms - Input & Validation** (CRITICAL) - Proper form implementation
5. **Forms - Error Handling** (HIGH) - Clear error recovery
6. **Forms - User Experience** (MEDIUM) - Advanced form patterns
7. **Animation & Motion** (CRITICAL) - Vestibular disorder considerations
8. **Performance & UX** (MEDIUM) - Load time and layout stability

---

# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. Accessibility - Semantic Structure (a11y)

**Impact:** CRITICAL
**Description:** Semantic HTML and ARIA provide the foundation for accessible interfaces. Proper structure enables assistive technologies to understand and navigate content, directly impacting users with disabilities.

## 2. Accessibility - Keyboard & Focus (a11y)

**Impact:** CRITICAL
**Description:** Keyboard navigation and focus management are essential for users who cannot use a mouse. Missing or broken keyboard support excludes a significant portion of users.

## 3. Accessibility - Visual & Color (a11y)

**Impact:** CRITICAL
**Description:** Color contrast, text sizing, and visual indicators ensure content is perceivable by users with visual impairments or color blindness.

## 4. Forms - Input & Validation (form)

**Impact:** CRITICAL
**Description:** Proper form implementation with clear labels, autocomplete, and validation reduces errors and improves completion rates. Poor forms are the #1 cause of user frustration.

## 5. Forms - Error Handling (form)

**Impact:** HIGH
**Description:** Clear, accessible error messages help users recover from mistakes quickly. Poor error handling leads to form abandonment.

## 6. Forms - User Experience (form)

**Impact:** MEDIUM
**Description:** Advanced form patterns like multi-step flows and inline validation improve the user experience and increase conversion rates.

## 7. Animation & Motion (motion)

**Impact:** HIGH
**Description:** Respecting prefers-reduced-motion is critical for users with vestibular disorders. Improper animations can cause nausea, dizziness, or seizures.

## 8. Performance & UX (perf)

**Impact:** MEDIUM
**Description:** Performance directly impacts user experience. Images without dimensions cause layout shifts, lazy loading improves initial load time.

---

---
title: Provide Meaningful Alt Text for Images
impact: CRITICAL
impactDescription: WCAG 2.1 Level A - Required for non-text content
tags: accessibility, images, alt-text, screen-readers
---

## Provide Meaningful Alt Text for Images

**Impact: CRITICAL (WCAG 2.1 Level A - Required for non-text content)**

Provide meaningful alternative text for images that conveys the purpose and content of the image to users who cannot see it.

## Bad Example

```html
<!-- Anti-pattern: Missing or poor alt text -->
<img src="hero.jpg">

<img src="logo.png" alt="logo">

<img src="team-photo.jpg" alt="image">

<img src="chart.png" alt="chart.png">

<img src="product.jpg" alt="Click here to buy now!">

<!-- Decorative image with alt text -->
<img src="decorative-border.png" alt="decorative border image">

<!-- Complex image with inadequate description -->
<img src="infographic.png" alt="infographic">

<!-- Icon with redundant alt -->
<a href="/search">
  <img src="search-icon.svg" alt="search icon">
  Search
</a>
```

## Good Example

```html
<!-- Correct approach: Meaningful, contextual alt text -->

<!-- Informative image: describe content and purpose -->
<img src="hero.jpg" alt="Team of developers collaborating around a whiteboard filled with code diagrams">

<!-- Logo: include company name -->
<img src="logo.png" alt="Acme Corporation">

<!-- Team photo: describe relevant details -->
<img src="team-photo.jpg" alt="Our 12-person development team at the 2024 company retreat in Colorado">

<!-- Chart: describe the data and insights -->
<img src="chart.png" alt="Line chart showing website traffic growth from 10,000 monthly visitors in January to 45,000 in December 2024">

<!-- Product image: describe the product -->
<img src="product.jpg" alt="Blue wireless headphones with cushioned ear cups and adjustable headband">

<!-- Decorative image: use empty alt -->
<img src="decorative-border.png" alt="" role="presentation">

<!-- Complex image: use figure and detailed description -->
<figure>
  <img src="infographic.png" alt="2024 Industry trends infographic (detailed description below)">
  <figcaption>
    <details>
      <summary>Detailed description of infographic</summary>
      <p>This infographic shows five key industry trends for 2024:</p>
      <ol>
        <li>AI adoption increased 45% year over year</li>
        <li>Remote work stabilized at 35% of workforce</li>
        <li>Cybersecurity spending grew by 20%</li>
        <li>Cloud migration reached 80% completion</li>
        <li>Sustainability initiatives in 60% of companies</li>
      </ol>
    </details>
  </figcaption>
</figure>

<!-- Icon with text: hide redundant icon -->
<a href="/search">
  <img src="search-icon.svg" alt="" aria-hidden="true">
  Search
</a>

<!-- Functional image: describe the action -->
<button>
  <img src="print-icon.svg" alt="Print this page">
</button>

<!-- Image as link: describe destination -->
<a href="/products/headphones">
  <img src="headphones-thumb.jpg" alt="View Blue Wireless Headphones product details">
</a>

<!-- Background image with important content -->
<div class="hero" style="background-image: url('hero-bg.jpg');" role="img" aria-label="Aerial view of San Francisco skyline at sunset">
  <h1>Welcome to San Francisco</h1>
</div>
```

## Why

Alt text is critical for accessibility and SEO:

1. **Screen Reader Users**: Alt text is read aloud by screen readers, providing context for visual content.

2. **Image Load Failures**: Alt text displays when images fail to load.

3. **Search Engines**: Alt text helps search engines understand image content.

4. **Cognitive Disabilities**: Alt text can help users understand the purpose of images.

5. **Legal Compliance**: WCAG requires text alternatives for non-text content.

Alt text guidelines:

**Informative Images:**
- Describe the content and function
- Be concise but complete (usually under 125 characters)
- Don't start with "Image of" or "Picture of"
- Include relevant details based on context

**Decorative Images:**
- Use empty alt (`alt=""`)
- Add `role="presentation"` for clarity
- CSS background images are inherently decorative

**Functional Images (buttons/links):**
- Describe the action, not the appearance
- "Submit form" not "Green button"

**Complex Images (charts, diagrams):**
- Provide brief alt text
- Offer detailed description via `<figcaption>`, link, or `aria-describedby`

**Images of Text:**
- Include all the text in the alt
- Better: use actual text instead of images

**Image Links:**
- Describe the destination
- "View product details" not "thumbnail image"

Testing tip: Ask yourself: "If I couldn't see this image, what information would I need?"

---

---
title: Add ARIA Labels to Interactive Elements
impact: CRITICAL
impactDescription: WCAG 2.1 Level A - Provides accessible names
tags: accessibility, aria, labels, screen-readers
---

## Add ARIA Labels to Interactive Elements

**Impact: CRITICAL (WCAG 2.1 Level A - Provides accessible names)**

Use ARIA (Accessible Rich Internet Applications) labels to provide accessible names and descriptions for elements that lack visible text or need additional context for assistive technologies.

## Bad Example

```html
<!-- Anti-pattern: Missing accessible names -->
<button>
  <svg viewBox="0 0 24 24">
    <path d="M19 13h-6v6h-2v-6H5v-2h6V5h2v6h6v2z"/>
  </svg>
</button>

<div class="search-box">
  <input type="text">
  <button>
    <img src="search-icon.png">
  </button>
</div>

<nav>
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/about">About</a></li>
  </ul>
</nav>
<nav>
  <ul>
    <li><a href="/docs">Documentation</a></li>
    <li><a href="/api">API Reference</a></li>
  </ul>
</nav>
```

## Good Example

```html
<!-- Correct approach: Using ARIA labels appropriately -->
<button aria-label="Add new item">
  <svg viewBox="0 0 24 24" aria-hidden="true">
    <path d="M19 13h-6v6h-2v-6H5v-2h6V5h2v6h6v2z"/>
  </svg>
</button>

<div class="search-box" role="search">
  <label for="site-search" class="visually-hidden">Search the site</label>
  <input type="text" id="site-search" aria-describedby="search-hint">
  <span id="search-hint" class="visually-hidden">Press Enter to search</span>
  <button aria-label="Submit search">
    <img src="search-icon.png" alt="" aria-hidden="true">
  </button>
</div>

<nav aria-label="Main navigation">
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/about">About</a></li>
  </ul>
</nav>
<nav aria-label="Documentation navigation">
  <ul>
    <li><a href="/docs">Documentation</a></li>
    <li><a href="/api">API Reference</a></li>
  </ul>
</nav>
```

## Why

ARIA labels are essential for accessibility because:

1. **Icon-only Buttons**: Buttons with only icons have no accessible name without ARIA labels. Screen reader users won't know what the button does.

2. **Distinguishing Repeated Elements**: When a page has multiple navigation regions, ARIA labels help users distinguish between them.

3. **Custom Components**: Interactive elements built with non-semantic HTML require ARIA to communicate their purpose and state.

4. **Additional Context**: `aria-describedby` provides supplementary information that helps users understand how to interact with an element.

Key ARIA label attributes:

- **`aria-label`**: Provides an accessible name directly on the element
- **`aria-labelledby`**: References another element's ID to use its text as the accessible name
- **`aria-describedby`**: References another element for additional descriptive text
- **`aria-hidden="true"`**: Hides decorative elements from assistive technologies

Best practices:

1. Prefer visible text over ARIA labels when possible
2. Don't duplicate visible text with aria-label
3. Keep labels concise but descriptive
4. Test with screen readers to verify the experience
5. Use `aria-hidden="true"` for decorative icons to prevent redundant announcements

---

---
title: Ensure Sufficient Color Contrast
impact: CRITICAL
impactDescription: WCAG 2.1 Level AA - 4.5:1 for text, 3:1 for large text
tags: accessibility, color, contrast, visual
---

## Ensure Sufficient Color Contrast

**Impact: CRITICAL (WCAG 2.1 Level AA - 4.5:1 for text, 3:1 for large text)**

Ensure sufficient color contrast between text and backgrounds to make content readable for users with low vision or color blindness.

## Bad Example

```css
/* Anti-pattern: Insufficient contrast ratios */
.hero-text {
  color: #999999; /* Gray text */
  background-color: #ffffff; /* White background */
  /* Contrast ratio: 2.85:1 - FAILS */
}

.subtle-text {
  color: #b3b3b3;
  background-color: #f5f5f5;
  /* Contrast ratio: 1.63:1 - FAILS */
}

.button-primary {
  color: #ffffff;
  background-color: #66b3ff; /* Light blue */
  /* Contrast ratio: 2.48:1 - FAILS */
}

.link {
  color: #6699ff; /* Light blue link on white */
  /* Contrast ratio: 3.03:1 - FAILS for normal text */
}

.placeholder {
  color: #cccccc; /* Placeholder text */
  /* Too light to read */
}
```

```html
<!-- Anti-pattern: Relying only on color -->
<p>Required fields are marked in <span style="color: red;">red</span>.</p>
<input type="text" style="border-color: red;">
```

## Good Example

```css
/* Correct approach: WCAG compliant contrast ratios */
.hero-text {
  color: #595959; /* Darker gray */
  background-color: #ffffff;
  /* Contrast ratio: 7:1 - PASSES AAA */
}

.body-text {
  color: #333333;
  background-color: #ffffff;
  /* Contrast ratio: 12.63:1 - PASSES AAA */
}

.button-primary {
  color: #ffffff;
  background-color: #0066cc; /* Darker blue */
  /* Contrast ratio: 7.05:1 - PASSES AAA */
}

.link {
  color: #0055aa; /* Darker blue link */
  text-decoration: underline;
  /* Contrast ratio: 7.28:1 - PASSES AAA */
}

.link:hover,
.link:focus {
  color: #003366;
  text-decoration: none;
  background-color: #e6f0ff;
}

/* Large text can have lower contrast (3:1 minimum) */
.large-heading {
  font-size: 24px;
  font-weight: bold;
  color: #666666;
  background-color: #ffffff;
  /* Contrast ratio: 5.74:1 - PASSES AA for large text */
}

/* Input placeholder with sufficient contrast */
.input::placeholder {
  color: #757575;
  /* Contrast ratio: 4.6:1 - PASSES AA */
}
```

```html
<!-- Correct approach: Color plus additional indicators -->
<p>Required fields are marked with an asterisk (*).</p>
<label for="email">
  Email <span aria-hidden="true">*</span>
  <span class="visually-hidden">required</span>
</label>
<input type="email" id="email" required aria-required="true">

<!-- Error state with icon and text, not just color -->
<div class="error-message" role="alert">
  <svg aria-hidden="true" class="error-icon"><!-- X icon --></svg>
  <span>Please enter a valid email address</span>
</div>
```

## Why

Adequate color contrast is crucial because:

1. **Low Vision**: Approximately 1 in 12 men and 1 in 200 women have some form of color vision deficiency.

2. **Aging Eyes**: Contrast sensitivity decreases with age. The elderly population often needs higher contrast.

3. **Environmental Factors**: Bright sunlight, glare, or low-quality screens can reduce perceived contrast.

4. **Legal Compliance**: WCAG guidelines require specific contrast ratios, and many laws reference WCAG.

WCAG contrast requirements:

- **Normal text (under 18pt or 14pt bold)**: 4.5:1 minimum (AA), 7:1 enhanced (AAA)
- **Large text (18pt+ or 14pt+ bold)**: 3:1 minimum (AA), 4.5:1 enhanced (AAA)
- **UI components and graphics**: 3:1 minimum against adjacent colors

Best practices:

- Use contrast checking tools during design and development
- Test with different types of color blindness simulators
- Never rely on color alone to convey information
- Provide alternative indicators (icons, patterns, text labels)
- Consider providing a high-contrast mode
- Test on different devices and in various lighting conditions

Recommended tools:
- WebAIM Contrast Checker
- Chrome DevTools Accessibility panel
- Stark (Figma/Sketch plugin)
- Color Oracle (color blindness simulator)

---

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

---

---
title: Manage Keyboard Focus Properly
impact: CRITICAL
impactDescription: WCAG 2.1 Level A - Focus order and visibility
tags: accessibility, keyboard, focus, navigation, modals
---

## Manage Keyboard Focus Properly

**Impact: CRITICAL (WCAG 2.1 Level A - Focus order and visibility)**

Manage keyboard focus intentionally to create a logical, predictable navigation experience. Focus should follow the user's expectations and never get lost or trapped.

## Bad Example

```html
<!-- Anti-pattern: Poor focus management -->
<div class="modal" style="display: block;">
  <div class="modal-content">
    <h2>Delete Item?</h2>
    <p>This action cannot be undone.</p>
    <button onclick="closeModal()">Cancel</button>
    <button onclick="deleteItem()">Delete</button>
  </div>
</div>
<!-- Focus stays on the trigger button behind the modal -->

<script>
function openModal() {
  document.querySelector('.modal').style.display = 'block';
  // No focus management - user is lost
}

function closeModal() {
  document.querySelector('.modal').style.display = 'none';
  // Focus doesn't return to trigger
}
</script>
```

```css
/* Anti-pattern: Removing focus indicator entirely */
*:focus {
  outline: none;
}
```

## Good Example

```html
<!-- Correct approach: Proper focus management -->
<button id="delete-trigger" onclick="openModal()">Delete Item</button>

<div class="modal"
     role="dialog"
     aria-modal="true"
     aria-labelledby="modal-title"
     hidden>
  <div class="modal-content">
    <h2 id="modal-title">Delete Item?</h2>
    <p>This action cannot be undone.</p>
    <button id="cancel-btn" onclick="closeModal()">Cancel</button>
    <button onclick="deleteItem()">Delete</button>
  </div>
</div>

<script>
let lastFocusedElement;

function openModal() {
  lastFocusedElement = document.activeElement;
  const modal = document.querySelector('.modal');
  modal.hidden = false;

  // Move focus to first focusable element
  document.getElementById('cancel-btn').focus();

  // Trap focus within modal
  modal.addEventListener('keydown', trapFocus);
}

function closeModal() {
  const modal = document.querySelector('.modal');
  modal.hidden = true;
  modal.removeEventListener('keydown', trapFocus);

  // Return focus to trigger element
  lastFocusedElement.focus();
}

function trapFocus(e) {
  if (e.key !== 'Tab') return;

  const focusableElements = modal.querySelectorAll(
    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
  );
  const firstElement = focusableElements[0];
  const lastElement = focusableElements[focusableElements.length - 1];

  if (e.shiftKey && document.activeElement === firstElement) {
    lastElement.focus();
    e.preventDefault();
  } else if (!e.shiftKey && document.activeElement === lastElement) {
    firstElement.focus();
    e.preventDefault();
  }
}
</script>
```

```css
/* Correct approach: Custom focus indicator */
*:focus {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}

/* Enhanced focus for better visibility */
*:focus-visible {
  outline: 3px solid #005fcc;
  outline-offset: 2px;
  box-shadow: 0 0 0 4px rgba(0, 95, 204, 0.3);
}
```

## Why

Proper focus management is critical for keyboard and screen reader users:

1. **Keyboard Navigation**: Users who can't use a mouse rely entirely on keyboard navigation. Focus must be visible and logical.

2. **Modal Dialogs**: Focus must be trapped within modals to prevent users from accidentally interacting with content behind the modal.

3. **Focus Restoration**: When closing modals or completing actions, focus should return to a logical position, usually the trigger element.

4. **Page Transitions**: After dynamic content loads or pages change in SPAs, focus should move to the new content or a logical starting point.

5. **Skip to Content**: Allow users to bypass repetitive navigation and jump to main content.

Focus management principles:

- Never remove focus indicators entirely
- Customize focus styles to match your design while maintaining visibility
- Use `tabindex="0"` to make custom elements focusable
- Use `tabindex="-1"` to make elements programmatically focusable but not in tab order
- Never use positive tabindex values as they disrupt natural document order
- Test focus flow by navigating your entire page using only the keyboard

---

---
title: Associate Labels with Form Inputs
impact: CRITICAL
impactDescription: WCAG 2.1 Level A - Info and Relationships
tags: accessibility, forms, labels, inputs
---

## Associate Labels with Form Inputs

**Impact: CRITICAL (WCAG 2.1 Level A - Info and Relationships)**

Every form input must have an associated label that clearly identifies its purpose. Labels are essential for screen reader users and improve usability for all users.

## Bad Example

```html
<!-- Anti-pattern: Missing or improper labels -->
<form>
  <!-- No label at all -->
  <input type="text" placeholder="Enter your name">

  <!-- Placeholder as label (disappears on input) -->
  <input type="email" placeholder="Email address">

  <!-- Label not associated with input -->
  <div>
    <span>Password</span>
    <input type="password">
  </div>

  <!-- Mismatched for/id -->
  <label for="username">Username</label>
  <input type="text" id="user-name">

  <!-- Non-label element used -->
  <p>Phone Number</p>
  <input type="tel">

  <!-- Label too far from input -->
  <label for="address">Address</label>
  <p>Enter your full mailing address including zip code</p>
  <div class="spacer"></div>
  <input type="text" id="address">
</form>
```

## Good Example

```html
<!-- Correct approach: Properly associated labels -->
<form>
  <!-- Explicit label association with for/id -->
  <div class="form-group">
    <label for="full-name">Full Name</label>
    <input type="text" id="full-name" name="fullName" autocomplete="name">
  </div>

  <!-- Implicit label (input inside label) -->
  <label class="form-group">
    Email Address
    <input type="email" name="email" autocomplete="email">
  </label>

  <!-- Label with required indicator -->
  <div class="form-group">
    <label for="password">
      Password
      <span aria-hidden="true">*</span>
      <span class="visually-hidden">(required)</span>
    </label>
    <input type="password" id="password" name="password"
           required aria-required="true"
           aria-describedby="password-requirements">
    <p id="password-requirements" class="helper-text">
      Must be at least 8 characters with one number
    </p>
  </div>

  <!-- Visually hidden label (when design requires no visible label) -->
  <div class="form-group search-box">
    <label for="search" class="visually-hidden">Search products</label>
    <input type="search" id="search" name="search"
           placeholder="Search..." autocomplete="off">
    <button type="submit" aria-label="Submit search">
      <svg aria-hidden="true"><!-- search icon --></svg>
    </button>
  </div>

  <!-- Fieldset for grouped inputs -->
  <fieldset>
    <legend>Shipping Address</legend>

    <div class="form-group">
      <label for="street">Street Address</label>
      <input type="text" id="street" name="street" autocomplete="street-address">
    </div>

    <div class="form-group">
      <label for="city">City</label>
      <input type="text" id="city" name="city" autocomplete="address-level2">
    </div>
  </fieldset>

  <!-- Radio buttons with fieldset/legend -->
  <fieldset>
    <legend>Preferred Contact Method</legend>

    <div class="radio-group">
      <input type="radio" id="contact-email" name="contact" value="email">
      <label for="contact-email">Email</label>
    </div>

    <div class="radio-group">
      <input type="radio" id="contact-phone" name="contact" value="phone">
      <label for="contact-phone">Phone</label>
    </div>

    <div class="radio-group">
      <input type="radio" id="contact-sms" name="contact" value="sms">
      <label for="contact-sms">Text Message</label>
    </div>
  </fieldset>

  <!-- Checkbox with label -->
  <div class="checkbox-group">
    <input type="checkbox" id="newsletter" name="newsletter">
    <label for="newsletter">Subscribe to our newsletter</label>
  </div>

  <!-- Multiple related inputs with single label -->
  <fieldset>
    <legend>Date of Birth</legend>
    <div class="date-inputs">
      <div>
        <label for="dob-month">Month</label>
        <select id="dob-month" name="dobMonth">
          <option value="">--</option>
          <option value="1">January</option>
          <!-- ... -->
        </select>
      </div>
      <div>
        <label for="dob-day">Day</label>
        <input type="text" id="dob-day" name="dobDay" maxlength="2">
      </div>
      <div>
        <label for="dob-year">Year</label>
        <input type="text" id="dob-year" name="dobYear" maxlength="4">
      </div>
    </div>
  </fieldset>
</form>

<style>
.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
</style>
```

## Why

Form labels are essential for multiple reasons:

1. **Screen Readers**: Labels are read aloud when inputs receive focus, telling users what information is expected.

2. **Click Target**: Clicking a label focuses/activates its associated input, improving usability for all users.

3. **Voice Control**: Users of voice recognition software use labels to identify and interact with inputs.

4. **Cognitive Accessibility**: Clear labels help users understand what information is needed.

5. **Legal Compliance**: WCAG requires programmatically associated labels.

Label best practices:

1. **Always Associate Labels**: Use `for`/`id` or wrap inputs in labels
2. **Position Consistently**: Labels above or to the left of inputs
3. **Use Placeholder Appropriately**: As hints, not replacements for labels
4. **Group Related Inputs**: Use `<fieldset>` and `<legend>`
5. **Indicate Required Fields**: Use both visual and programmatic indicators
6. **Add Helper Text**: Use `aria-describedby` for additional instructions
7. **Keep Labels Visible**: Hidden labels should be last resort
8. **Be Descriptive**: "Email Address" not just "Email"
9. **Avoid Instructions in Labels**: "Enter your email" should be "Email Address"

---

---
title: Maintain Proper Heading Hierarchy
impact: CRITICAL
impactDescription: WCAG 2.1 Level A - Info and Relationships
tags: accessibility, headings, structure, navigation
---

## Maintain Proper Heading Hierarchy

**Impact: CRITICAL (WCAG 2.1 Level A - Info and Relationships)**

Use heading elements (h1-h6) in a logical, hierarchical order to create a clear document outline. Proper heading structure is essential for screen reader navigation and SEO.

## Bad Example

```html
<!-- Anti-pattern: Broken heading hierarchy -->
<body>
  <!-- Skipping h1 -->
  <h2>Welcome to Our Store</h2>

  <div class="hero">
    <!-- Skipping levels (h2 to h4) -->
    <h4>Shop the Latest Collection</h4>
  </div>

  <section>
    <!-- Using h1 for styling purposes -->
    <h1 class="small-heading">Featured Products</h1>

    <div class="product">
      <!-- Heading levels chosen for styling, not structure -->
      <h5>Product Name</h5>
      <h6>$29.99</h6>
    </div>
  </section>

  <aside>
    <!-- Multiple h1s on the page -->
    <h1>Special Offers</h1>
  </aside>

  <!-- Using heading for non-heading content -->
  <h3>© 2024 Our Company</h3>
</body>
```

## Good Example

```html
<!-- Correct approach: Logical heading hierarchy -->
<body>
  <header>
    <a href="/" aria-label="Home">
      <img src="logo.svg" alt="Acme Store">
    </a>
  </header>

  <!-- Single h1 for main page title -->
  <main>
    <h1>Welcome to Acme Store</h1>

    <section aria-labelledby="hero-heading">
      <h2 id="hero-heading">Shop the Latest Collection</h2>
      <p>Discover our new arrivals for the season.</p>
    </section>

    <section aria-labelledby="featured-heading">
      <h2 id="featured-heading">Featured Products</h2>

      <article class="product">
        <h3>Wireless Headphones</h3>
        <p class="price">$29.99</p>
        <p>High-quality audio with 20-hour battery life.</p>
      </article>

      <article class="product">
        <h3>Smart Watch</h3>
        <p class="price">$199.99</p>
        <p>Track your fitness and stay connected.</p>

        <!-- Sub-sections within product -->
        <section>
          <h4>Key Features</h4>
          <ul>
            <li>Heart rate monitor</li>
            <li>GPS tracking</li>
            <li>Water resistant</li>
          </ul>
        </section>

        <section>
          <h4>Customer Reviews</h4>
          <div class="review">
            <h5>Great product!</h5>
            <p>Exceeded my expectations...</p>
          </div>
        </section>
      </article>
    </section>

    <section aria-labelledby="categories-heading">
      <h2 id="categories-heading">Shop by Category</h2>

      <div class="category">
        <h3>Electronics</h3>
        <ul>
          <li><a href="/electronics/phones">Phones</a></li>
          <li><a href="/electronics/tablets">Tablets</a></li>
        </ul>
      </div>

      <div class="category">
        <h3>Clothing</h3>
        <ul>
          <li><a href="/clothing/mens">Men's</a></li>
          <li><a href="/clothing/womens">Women's</a></li>
        </ul>
      </div>
    </section>
  </main>

  <aside aria-labelledby="offers-heading">
    <h2 id="offers-heading">Special Offers</h2>
    <p>Get 20% off your first order!</p>
  </aside>

  <footer>
    <!-- Non-heading content styled differently -->
    <p><small>&copy; 2024 Acme Store. All rights reserved.</small></p>
  </footer>
</body>

<style>
/* Style headings independently of their level */
.product h3 {
  font-size: 1.25rem;
  font-weight: 600;
}

.product h4 {
  font-size: 1rem;
  font-weight: 500;
  color: #666;
}

/* Utility for visually styling any element as a heading */
.looks-like-h2 {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 1rem;
}
</style>
```

## Why

Proper heading hierarchy is fundamental for accessibility:

1. **Screen Reader Navigation**: Users navigate by headings (H key). They rely on the hierarchy to understand document structure and jump to sections.

2. **Document Outline**: Headings create an outline that helps all users understand content organization.

3. **SEO**: Search engines use heading hierarchy to understand page structure and importance.

4. **Cognitive Accessibility**: Clear structure helps users with cognitive disabilities process information.

5. **WCAG Compliance**: Success Criterion 1.3.1 requires proper heading structure.

Heading hierarchy rules:

1. **One h1 per Page**: Use h1 for the main page title or topic
2. **Don't Skip Levels**: Go h1 → h2 → h3 (not h1 → h3)
3. **Nest Properly**: h3 should be inside h2 section, etc.
4. **Semantic, Not Visual**: Choose heading level based on structure, not appearance
5. **Style with CSS**: Use CSS to control visual appearance of any heading level
6. **Headings for Structure**: Don't use headings just for bold text

Screen reader heading navigation:
- **H key**: Next heading
- **Shift+H**: Previous heading
- **1-6 keys**: Jump to specific heading level
- **Insert+F6** (JAWS): Heading list

Testing your heading structure:
- Use browser extensions like HeadingsMap
- Use screen reader heading navigation
- Review the document outline
- Check WAVE accessibility tool

Remember: The visual size and style of headings should be controlled by CSS, not by choosing inappropriate heading levels.

---

---
title: Ensure Full Keyboard Navigation
impact: CRITICAL
impactDescription: WCAG 2.1 Level A - Required for keyboard-only users
tags: accessibility, keyboard, focus, navigation
---

## Ensure Full Keyboard Navigation

**Impact: CRITICAL (WCAG 2.1 Level A - Required for keyboard-only users)**

Ensure all interactive elements and functionality are fully accessible via keyboard. Users should be able to navigate, activate, and interact with all features without requiring a mouse.

## Bad Example

```html
<!-- Anti-pattern: Click-only interactions -->
<div class="card" onclick="showDetails()">
  <img src="product.jpg">
  <span class="title">Product Name</span>
</div>

<span class="link" onclick="navigate('/page')">Go to page</span>

<div class="dropdown">
  <div class="trigger" onclick="toggleMenu()">Menu</div>
  <div class="menu">
    <div class="item" onclick="selectItem(1)">Option 1</div>
    <div class="item" onclick="selectItem(2)">Option 2</div>
    <div class="item" onclick="selectItem(3)">Option 3</div>
  </div>
</div>

<div class="slider" onmousedown="startDrag()">
  <div class="handle"></div>
</div>
```

## Good Example

```html
<!-- Correct approach: Full keyboard support -->
<article class="card"
         tabindex="0"
         role="button"
         onclick="showDetails()"
         onkeydown="handleCardKey(event)">
  <img src="product.jpg" alt="Product Name - Blue Widget">
  <h3 class="title">Product Name</h3>
</article>

<a href="/page" class="link">Go to page</a>

<div class="dropdown">
  <button class="trigger"
          aria-expanded="false"
          aria-haspopup="menu"
          onclick="toggleMenu()"
          onkeydown="handleTriggerKey(event)">
    Menu
  </button>
  <ul class="menu" role="menu" hidden>
    <li role="menuitem" tabindex="-1" onkeydown="handleMenuKey(event, 0)">Option 1</li>
    <li role="menuitem" tabindex="-1" onkeydown="handleMenuKey(event, 1)">Option 2</li>
    <li role="menuitem" tabindex="-1" onkeydown="handleMenuKey(event, 2)">Option 3</li>
  </ul>
</div>

<div class="slider"
     role="slider"
     tabindex="0"
     aria-valuemin="0"
     aria-valuemax="100"
     aria-valuenow="50"
     aria-label="Volume"
     onkeydown="handleSliderKey(event)">
  <div class="handle"></div>
</div>

<script>
function handleCardKey(event) {
  if (event.key === 'Enter' || event.key === ' ') {
    event.preventDefault();
    showDetails();
  }
}

function handleTriggerKey(event) {
  switch (event.key) {
    case 'ArrowDown':
    case 'Enter':
    case ' ':
      event.preventDefault();
      openMenu();
      focusFirstItem();
      break;
    case 'Escape':
      closeMenu();
      break;
  }
}

function handleMenuKey(event, index) {
  switch (event.key) {
    case 'ArrowDown':
      event.preventDefault();
      focusNextItem(index);
      break;
    case 'ArrowUp':
      event.preventDefault();
      focusPrevItem(index);
      break;
    case 'Enter':
    case ' ':
      event.preventDefault();
      selectItem(index);
      closeMenu();
      break;
    case 'Escape':
      closeMenu();
      focusTrigger();
      break;
  }
}

function handleSliderKey(event) {
  const slider = event.target;
  let value = parseInt(slider.getAttribute('aria-valuenow'));

  switch (event.key) {
    case 'ArrowRight':
    case 'ArrowUp':
      event.preventDefault();
      value = Math.min(100, value + 1);
      break;
    case 'ArrowLeft':
    case 'ArrowDown':
      event.preventDefault();
      value = Math.max(0, value - 1);
      break;
    case 'PageUp':
      event.preventDefault();
      value = Math.min(100, value + 10);
      break;
    case 'PageDown':
      event.preventDefault();
      value = Math.max(0, value - 10);
      break;
    case 'Home':
      event.preventDefault();
      value = 0;
      break;
    case 'End':
      event.preventDefault();
      value = 100;
      break;
  }

  slider.setAttribute('aria-valuenow', value);
  updateSliderPosition(value);
}
</script>
```

## Why

Keyboard accessibility is essential because:

1. **Motor Disabilities**: Many users cannot use a mouse due to motor impairments and rely on keyboard or switch devices.

2. **Screen Reader Users**: Screen reader users primarily navigate using keyboard commands.

3. **Power Users**: Many users prefer keyboard navigation for efficiency.

4. **Temporary Situations**: Broken mouse, using a laptop trackpad, or repetitive strain injury.

5. **Legal Requirements**: WCAG and accessibility laws require keyboard accessibility.

Standard keyboard patterns:

- **Tab/Shift+Tab**: Move between focusable elements
- **Enter/Space**: Activate buttons and links
- **Arrow keys**: Navigate within components (menus, tabs, sliders)
- **Escape**: Close modals, menus, or cancel operations
- **Home/End**: Jump to first/last item in a list
- **Page Up/Down**: Scroll or move by larger increments

Best practices:

- Use native HTML elements when possible (they have built-in keyboard support)
- Implement standard keyboard patterns for custom components
- Test your entire application using only the keyboard
- Provide visible focus indicators
- Avoid keyboard traps (except in modals)
- Document any non-standard keyboard shortcuts

---

---
title: Use ARIA Live Regions for Dynamic Content
impact: HIGH
impactDescription: WCAG 2.1 Level A - Status messages
tags: accessibility, aria, live-regions, dynamic-content
---

## Use ARIA Live Regions for Dynamic Content

**Impact: HIGH (WCAG 2.1 Level A - Status messages)**

Use ARIA live regions to announce dynamic content changes to screen reader users. Live regions ensure that updates happening outside the user's current focus are communicated appropriately.

## Bad Example

```html
<!-- Anti-pattern: Dynamic updates not announced -->

<!-- Cart count updates silently -->
<div class="cart-count">3</div>

<!-- Loading state not communicated -->
<div class="loading-spinner" style="display: none;">
  <div class="spinner"></div>
</div>

<!-- Toast notification appears but isn't announced -->
<div class="toast success">
  Your changes have been saved!
</div>

<!-- Search results update without announcement -->
<div class="search-results">
  <p>Showing 25 results for "widgets"</p>
  <!-- results -->
</div>

<!-- Form submission feedback not announced -->
<script>
function submitForm() {
  // ... submit logic
  document.querySelector('.success-message').style.display = 'block';
}
</script>
<div class="success-message" style="display: none;">
  Form submitted successfully!
</div>
```

## Good Example

```html
<!-- Correct approach: Dynamic updates properly announced -->

<!-- Cart count with live region -->
<div class="cart">
  <span class="visually-hidden" aria-live="polite" aria-atomic="true">
    Shopping cart: <span id="cart-count-announcement">3 items</span>
  </span>
  <button aria-label="Shopping cart, 3 items">
    <svg aria-hidden="true"><!-- cart icon --></svg>
    <span class="cart-count" aria-hidden="true">3</span>
  </button>
</div>

<!-- Loading state with status updates -->
<div class="content-area">
  <div id="loading-status"
       role="status"
       aria-live="polite"
       class="visually-hidden">
  </div>
  <div class="loading-spinner" hidden>
    <div class="spinner" aria-hidden="true"></div>
    <span class="visually-hidden">Loading content, please wait</span>
  </div>
  <div id="content">
    <!-- Dynamic content here -->
  </div>
</div>

<!-- Toast notification with proper live region -->
<div id="notification-area"
     role="alert"
     aria-live="assertive"
     aria-atomic="true">
</div>

<!-- Search results with polite announcement -->
<div class="search-container">
  <div id="search-status"
       role="status"
       aria-live="polite"
       aria-atomic="true"
       class="visually-hidden">
  </div>
  <div class="search-results" aria-labelledby="results-heading">
    <h2 id="results-heading">Search Results</h2>
    <p id="results-summary">Showing 25 results for "widgets"</p>
    <!-- results -->
  </div>
</div>

<!-- Form with proper feedback -->
<form id="contact-form">
  <div id="form-status"
       role="status"
       aria-live="polite"
       aria-atomic="true">
  </div>
  <!-- form fields -->
  <button type="submit">Submit</button>
</form>

<script>
// Cart update with announcement
function updateCart(count) {
  document.getElementById('cart-count-announcement').textContent =
    `${count} item${count !== 1 ? 's' : ''}`;
}

// Loading with status updates
function loadContent() {
  const status = document.getElementById('loading-status');
  const spinner = document.querySelector('.loading-spinner');

  status.textContent = 'Loading content, please wait...';
  spinner.hidden = false;

  fetch('/api/content')
    .then(response => response.json())
    .then(data => {
      spinner.hidden = true;
      document.getElementById('content').innerHTML = data.html;
      status.textContent = 'Content loaded successfully';
    })
    .catch(error => {
      spinner.hidden = true;
      status.textContent = 'Failed to load content. Please try again.';
    });
}

// Toast notification
function showToast(message, type = 'info') {
  const area = document.getElementById('notification-area');
  area.innerHTML = `
    <div class="toast ${type}">
      <svg aria-hidden="true" class="${type}-icon"><!-- icon --></svg>
      ${message}
    </div>
  `;

  // Auto-dismiss after delay
  setTimeout(() => {
    area.innerHTML = '';
  }, 5000);
}

// Search results announcement
function updateSearchResults(query, count) {
  const status = document.getElementById('search-status');
  status.textContent = `Showing ${count} results for "${query}"`;
}

// Form submission with feedback
document.getElementById('contact-form').addEventListener('submit', async (e) => {
  e.preventDefault();
  const status = document.getElementById('form-status');

  status.textContent = 'Submitting form...';

  try {
    await submitForm();
    status.textContent = 'Form submitted successfully! We will contact you within 24 hours.';
  } catch (error) {
    status.textContent = 'Form submission failed. Please try again or contact support.';
  }
});
</script>
```

## Why

Live regions are essential for dynamic web applications:

1. **Screen Reader Awareness**: Without live regions, screen reader users have no way of knowing when content changes elsewhere on the page.

2. **Async Operations**: Modern applications frequently update content via AJAX. These updates need to be communicated.

3. **User Feedback**: Loading states, form submissions, and notifications must reach all users.

4. **Focus Independence**: Users shouldn't need to manually check for updates.

Live region attributes:

- **`aria-live="polite"`**: Announces when user is idle (most common)
- **`aria-live="assertive"`**: Interrupts immediately (use sparingly)
- **`role="status"`**: Equivalent to `aria-live="polite"` with `aria-atomic="true"`
- **`role="alert"`**: Equivalent to `aria-live="assertive"`
- **`aria-atomic="true"`**: Announces entire region on change
- **`aria-relevant`**: What types of changes to announce (additions, removals, text)

Best practices:

1. **Prefer Polite**: Use assertive only for critical, time-sensitive information
2. **Pre-existing Regions**: Create live regions in initial HTML, not dynamically
3. **Keep Concise**: Brief, meaningful announcements
4. **Avoid Overuse**: Too many announcements overwhelm users
5. **Update Content, Not Container**: Change the text inside the live region
6. **Test with Screen Readers**: Behavior varies across screen readers
7. **Combine with Visual**: Live regions supplement visual feedback

---

---
title: Optimize for Screen Reader Compatibility
impact: CRITICAL
impactDescription: WCAG 2.1 Level A - Perceivable and operable
tags: accessibility, screen-readers, semantic-html, aria
---

## Optimize for Screen Reader Compatibility

**Impact: CRITICAL (WCAG 2.1 Level A - Perceivable and operable)**

Design and code with screen reader users in mind. Screen readers convert visual content to audio output, requiring careful attention to how information is structured and announced.

## Bad Example

```html
<!-- Anti-pattern: Poor screen reader experience -->
<div class="breadcrumb">
  Home > Products > Electronics > Phones
</div>

<table>
  <tr>
    <td>Name</td>
    <td>Price</td>
    <td>Stock</td>
  </tr>
  <tr>
    <td>Widget</td>
    <td>$9.99</td>
    <td>✓</td>
  </tr>
</table>

<div class="rating">
  ★★★★☆
</div>

<button class="close-btn">×</button>

<div class="price">
  <span class="currency">$</span>
  <span class="dollars">29</span>
  <span class="cents">99</span>
</div>

<img src="chart.png">
```

## Good Example

```html
<!-- Correct approach: Screen reader optimized -->
<nav aria-label="Breadcrumb">
  <ol class="breadcrumb">
    <li><a href="/">Home</a></li>
    <li><a href="/products">Products</a></li>
    <li><a href="/products/electronics">Electronics</a></li>
    <li aria-current="page">Phones</li>
  </ol>
</nav>

<table>
  <caption>Product Inventory</caption>
  <thead>
    <tr>
      <th scope="col">Name</th>
      <th scope="col">Price</th>
      <th scope="col">In Stock</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Widget</th>
      <td>$9.99</td>
      <td>
        <span aria-label="Yes, in stock">✓</span>
      </td>
    </tr>
  </tbody>
</table>

<div class="rating" role="img" aria-label="4 out of 5 stars">
  <span aria-hidden="true">★★★★☆</span>
</div>

<button class="close-btn" aria-label="Close dialog">
  <span aria-hidden="true">×</span>
</button>

<div class="price">
  <span class="visually-hidden">Price: $29.99</span>
  <span aria-hidden="true">
    <span class="currency">$</span>
    <span class="dollars">29</span>
    <span class="cents">99</span>
  </span>
</div>

<figure>
  <img src="chart.png" alt="Sales chart showing 25% growth in Q4">
  <figcaption>
    Quarterly sales data for 2024. Q4 shows the highest growth at 25%.
  </figcaption>
</figure>

<!-- Visually hidden utility class -->
<style>
.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
</style>
```

## Why

Screen reader compatibility is essential because:

1. **Blind Users**: Screen readers are the primary way blind users access web content. Poor implementation creates an unusable experience.

2. **Information Loss**: Visual-only information (icons, symbols, colors) is completely lost without proper alternatives.

3. **Context**: Screen reader users hear content linearly and need proper structure to understand relationships.

4. **Navigation**: Users rely on headings, landmarks, and links to navigate efficiently.

Screen reader best practices:

1. **Use Semantic HTML**: Native elements have built-in accessibility
2. **Proper Heading Structure**: Use h1-h6 in logical order
3. **Meaningful Link Text**: "Read more about pricing" not "Click here"
4. **Table Structure**: Use `<th>`, `scope`, and `<caption>`
5. **Alternative Text**: Describe images meaningfully
6. **Form Labels**: Every input needs an associated label
7. **Live Regions**: Announce dynamic content changes
8. **Hide Decorative Content**: Use `aria-hidden="true"`
9. **Provide Context**: Use `aria-label` and `aria-describedby`
10. **Test with Screen Readers**: NVDA, JAWS, VoiceOver, TalkBack

Common screen reader commands users rely on:

- Navigate by headings (H key)
- Navigate by landmarks (D key)
- List all links (Insert+F7 in JAWS)
- Navigate tables (Ctrl+Alt+Arrow keys)
- Read from current position (Insert+Down Arrow)

Testing tip: Try using your site with your monitor turned off using only a screen reader to understand the experience.

---

---
title: Use Semantic HTML Elements
impact: CRITICAL
impactDescription: WCAG 2.1 Level A - Foundation for accessibility
tags: accessibility, semantic-html, screen-readers, seo
---

## Use Semantic HTML Elements

**Impact: CRITICAL (WCAG 2.1 Level A - Foundation for accessibility)**

## Why It Matters

Semantic HTML provides meaning to content, enabling screen readers and assistive technologies to understand page structure. It improves SEO, maintainability, and accessibility without extra effort.

## Incorrect

```tsx
// ❌ Div soup - no semantic meaning
<div className="header">
  <div className="logo">Logo</div>
  <div className="nav">
    <div onClick={handleHome}>Home</div>
    <div onClick={handleAbout}>About</div>
  </div>
</div>

<div className="main">
  <div className="article">
    <div className="title">Article Title</div>
    <div className="content">Article content...</div>
  </div>
  <div className="sidebar">Related links</div>
</div>

<div className="footer">
  Copyright 2024
</div>
```

**Problems:**
- Screen readers can't identify page structure
- No keyboard navigation for "clickable" divs
- Search engines can't understand content hierarchy
- Assistive technology can't navigate sections

## Correct

```tsx
// ✅ Semantic HTML - meaningful structure
<header>
  <a href="/" aria-label="Home">Logo</a>
  <nav aria-label="Main navigation">
    <ul>
      <li><a href="/">Home</a></li>
      <li><a href="/about">About</a></li>
    </ul>
  </nav>
</header>

<main>
  <article>
    <h1>Article Title</h1>
    <p>Article content...</p>
  </article>
  <aside aria-label="Related content">
    <h2>Related Links</h2>
    <ul>
      <li><a href="/related-1">Related Article 1</a></li>
    </ul>
  </aside>
</main>

<footer>
  <p>Copyright 2024</p>
</footer>
```

## Semantic Elements Reference

| Element | Purpose | Use For |
|---------|---------|---------|
| `<header>` | Introductory content | Page/section headers |
| `<nav>` | Navigation links | Main nav, breadcrumbs |
| `<main>` | Main content | Primary page content (one per page) |
| `<article>` | Self-contained content | Blog posts, news articles |
| `<section>` | Thematic grouping | Chapters, tabs, grouped content |
| `<aside>` | Tangentially related | Sidebars, pull quotes |
| `<footer>` | Footer content | Copyright, links, contact |
| `<figure>` | Self-contained media | Images with captions |
| `<figcaption>` | Caption for figure | Image/chart descriptions |
| `<time>` | Date/time | Dates, times, durations |
| `<address>` | Contact information | Author/organization contact |
| `<mark>` | Highlighted text | Search results highlighting |

## Interactive Elements

```tsx
// ❌ Div with click handler - not accessible
<div onClick={handleClick} className="button">
  Click me
</div>

// ✅ Button element - accessible by default
<button onClick={handleClick}>
  Click me
</button>

// ❌ Span as link
<span onClick={() => navigate('/about')} className="link">
  About
</span>

// ✅ Anchor element
<a href="/about">About</a>

// ✅ Link component (React Router/Next.js)
<Link href="/about">About</Link>
```

## Headings Hierarchy

```tsx
// ❌ Skipping heading levels
<h1>Page Title</h1>
<h3>Section Title</h3>  {/* Skipped h2 */}
<h5>Subsection</h5>     {/* Skipped h4 */}

// ✅ Proper heading hierarchy
<h1>Page Title</h1>
<h2>Section Title</h2>
<h3>Subsection Title</h3>
<h2>Another Section</h2>
<h3>Its Subsection</h3>
```

## Lists

```tsx
// ❌ Not using lists for list content
<div className="menu">
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</div>

// ✅ Use appropriate list elements
<ul>  {/* Unordered list */}
  <li>Item 1</li>
  <li>Item 2</li>
</ul>

<ol>  {/* Ordered list */}
  <li>Step 1</li>
  <li>Step 2</li>
</ol>

<dl>  {/* Description list */}
  <dt>Term</dt>
  <dd>Definition</dd>
</dl>
```

## Tables

```tsx
// ✅ Accessible table
<table>
  <caption>Monthly Sales Data</caption>
  <thead>
    <tr>
      <th scope="col">Month</th>
      <th scope="col">Sales</th>
      <th scope="col">Revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">January</th>
      <td>100</td>
      <td>$10,000</td>
    </tr>
  </tbody>
</table>
```

## Common Patterns

```tsx
// Card component
<article className="card">
  <header>
    <h3>Card Title</h3>
  </header>
  <p>Card content...</p>
  <footer>
    <a href="/read-more">Read more</a>
  </footer>
</article>

// Search form
<search>  {/* HTML5.2 element, or use role="search" */}
  <form role="search">
    <label htmlFor="search">Search</label>
    <input type="search" id="search" name="q" />
    <button type="submit">Search</button>
  </form>
</search>
```

## Benefits

- Screen readers announce page structure
- Keyboard users can navigate by landmarks
- Better SEO indexing
- Easier to style with CSS
- Future-proof and maintainable

---

---
title: Provide Skip Links for Navigation
impact: HIGH
impactDescription: WCAG 2.1 Level A - Bypass blocks
tags: accessibility, navigation, keyboard, skip-links
---

## Provide Skip Links for Navigation

**Impact: HIGH (WCAG 2.1 Level A - Bypass blocks)**

Provide skip links to allow keyboard and screen reader users to bypass repetitive content and navigate directly to main content areas.

## Bad Example

```html
<!-- Anti-pattern: No skip links -->
<!DOCTYPE html>
<html>
<head>
  <title>My Website</title>
</head>
<body>
  <header>
    <nav>
      <a href="/">Home</a>
      <a href="/about">About</a>
      <a href="/services">Services</a>
      <a href="/products">Products</a>
      <a href="/portfolio">Portfolio</a>
      <a href="/blog">Blog</a>
      <a href="/contact">Contact</a>
      <!-- 20+ more links in mega menu -->
    </nav>
  </header>
  <main>
    <h1>Welcome</h1>
    <!-- User must tab through ALL nav links to reach this content -->
  </main>
</body>
</html>
```

## Good Example

```html
<!-- Correct approach: Comprehensive skip links -->
<!DOCTYPE html>
<html>
<head>
  <title>My Website</title>
  <style>
    .skip-links {
      position: absolute;
      top: 0;
      left: 0;
      z-index: 9999;
    }

    .skip-link {
      position: absolute;
      left: -10000px;
      top: auto;
      width: 1px;
      height: 1px;
      overflow: hidden;
      background: #000;
      color: #fff;
      padding: 1rem;
      text-decoration: none;
      font-weight: bold;
    }

    .skip-link:focus {
      position: fixed;
      top: 0;
      left: 0;
      width: auto;
      height: auto;
      overflow: visible;
      outline: 3px solid #005fcc;
      outline-offset: 2px;
    }

    /* Smooth scroll for skip link targets */
    html {
      scroll-behavior: smooth;
    }

    /* Ensure skip link targets are focusable */
    [id]:target {
      scroll-margin-top: 1rem;
    }
  </style>
</head>
<body>
  <div class="skip-links">
    <a href="#main-content" class="skip-link">Skip to main content</a>
    <a href="#main-nav" class="skip-link">Skip to navigation</a>
    <a href="#search" class="skip-link">Skip to search</a>
    <a href="#footer" class="skip-link">Skip to footer</a>
  </div>

  <header>
    <form id="search" role="search" tabindex="-1">
      <label for="search-input" class="visually-hidden">Search</label>
      <input type="search" id="search-input" placeholder="Search...">
      <button type="submit">Search</button>
    </form>

    <nav id="main-nav" aria-label="Main navigation" tabindex="-1">
      <a href="/">Home</a>
      <a href="/about">About</a>
      <a href="/services">Services</a>
      <a href="/products">Products</a>
      <a href="/portfolio">Portfolio</a>
      <a href="/blog">Blog</a>
      <a href="/contact">Contact</a>
    </nav>
  </header>

  <main id="main-content" tabindex="-1">
    <h1>Welcome to Our Website</h1>
    <p>Main content starts here...</p>

    <!-- For long pages, provide in-page skip links -->
    <nav aria-label="Page sections">
      <h2>On this page</h2>
      <ul>
        <li><a href="#section-1">Introduction</a></li>
        <li><a href="#section-2">Features</a></li>
        <li><a href="#section-3">Pricing</a></li>
      </ul>
    </nav>

    <section id="section-1" tabindex="-1">
      <h2>Introduction</h2>
      <!-- Content -->
    </section>

    <section id="section-2" tabindex="-1">
      <h2>Features</h2>
      <!-- Content -->
    </section>

    <section id="section-3" tabindex="-1">
      <h2>Pricing</h2>
      <!-- Content -->
    </section>
  </main>

  <footer id="footer" tabindex="-1">
    <p>&copy; 2024 My Website</p>
  </footer>
</body>
</html>
```

## Why

Skip links are a WCAG requirement and essential for usability:

1. **Keyboard Efficiency**: Without skip links, keyboard users must tab through every navigation link on every page load to reach main content.

2. **Screen Reader Navigation**: While screen reader users can navigate by headings, skip links provide a direct, consistent mechanism.

3. **Repetitive Content**: Headers, navigation, sidebars appear on every page. Users shouldn't have to traverse them repeatedly.

4. **Motor Disabilities**: Reducing the number of key presses helps users with motor impairments.

5. **WCAG Requirement**: WCAG 2.1 Success Criterion 2.4.1 requires a mechanism to bypass repeated content.

Skip link best practices:

1. **First Focusable Element**: Skip link should be the very first focusable element
2. **Visible on Focus**: Hidden until focused, then prominently displayed
3. **Multiple Skip Links**: Consider links to main content, navigation, search, footer
4. **Target Focusable**: Add `tabindex="-1"` to skip link targets for browser compatibility
5. **Clear Labels**: "Skip to main content" is clearer than "Skip navigation"
6. **Consistent Placement**: Same location on every page
7. **High Contrast**: Ensure skip link is visible when focused
8. **Table of Contents**: For long pages, provide in-page navigation

Note: Some browsers don't move focus correctly to skip link targets without `tabindex="-1"` on the target element.

---

---
title: Use Autocomplete Attributes for Forms
impact: CRITICAL
impactDescription: WCAG 2.1 Level AA - Identify input purpose
tags: forms, autocomplete, accessibility, ux
---

## Use Autocomplete Attributes for Forms

**Impact: CRITICAL (WCAG 2.1 Level AA - Identify input purpose)**

## Why It Matters

The `autocomplete` attribute enables browsers and password managers to autofill forms correctly. Proper autocomplete improves user experience, reduces errors, and is required for WCAG 1.3.5 compliance.

## Incorrect

```tsx
// ❌ No autocomplete attributes
<form>
  <input type="text" name="name" />
  <input type="text" name="email" />
  <input type="password" name="password" />
</form>

// ❌ Autocomplete disabled (frustrating for users)
<input type="email" autoComplete="off" />
```

## Correct

```tsx
// ✅ Proper autocomplete attributes
<form>
  {/* Name fields */}
  <input type="text" name="name" autoComplete="name" />
  <input type="text" name="firstName" autoComplete="given-name" />
  <input type="text" name="lastName" autoComplete="family-name" />

  {/* Contact fields */}
  <input type="email" name="email" autoComplete="email" />
  <input type="tel" name="phone" autoComplete="tel" />

  {/* Address fields */}
  <input type="text" name="address" autoComplete="street-address" />
  <input type="text" name="city" autoComplete="address-level2" />
  <input type="text" name="state" autoComplete="address-level1" />
  <input type="text" name="zip" autoComplete="postal-code" />
  <input type="text" name="country" autoComplete="country-name" />

  {/* Payment fields */}
  <input type="text" name="ccName" autoComplete="cc-name" />
  <input type="text" name="ccNumber" autoComplete="cc-number" />
  <input type="text" name="ccExp" autoComplete="cc-exp" />
  <input type="text" name="ccCsc" autoComplete="cc-csc" />
</form>
```

## Login Form

```tsx
// ✅ Login form with proper autocomplete
<form>
  <div>
    <label htmlFor="username">Username or Email</label>
    <input
      id="username"
      type="email"
      name="email"
      autoComplete="username"  // Tells password managers this is the identifier
    />
  </div>

  <div>
    <label htmlFor="password">Password</label>
    <input
      id="password"
      type="password"
      name="password"
      autoComplete="current-password"  // For existing password
    />
  </div>

  <button type="submit">Sign In</button>
</form>
```

## Registration Form

```tsx
// ✅ Registration form
<form>
  <div>
    <label htmlFor="email">Email</label>
    <input
      id="email"
      type="email"
      name="email"
      autoComplete="email"
    />
  </div>

  <div>
    <label htmlFor="newPassword">Password</label>
    <input
      id="newPassword"
      type="password"
      name="password"
      autoComplete="new-password"  // For new password creation
    />
  </div>

  <div>
    <label htmlFor="confirmPassword">Confirm Password</label>
    <input
      id="confirmPassword"
      type="password"
      name="confirmPassword"
      autoComplete="new-password"
    />
  </div>

  <button type="submit">Create Account</button>
</form>
```

## One-Time Codes (OTP)

```tsx
// ✅ OTP input
<div>
  <label htmlFor="otp">Verification Code</label>
  <input
    id="otp"
    type="text"
    inputMode="numeric"
    pattern="[0-9]*"
    autoComplete="one-time-code"  // Triggers SMS autofill on mobile
    maxLength={6}
  />
</div>
```

## Common Autocomplete Values

| Value | Use For |
|-------|---------|
| `name` | Full name |
| `given-name` | First name |
| `family-name` | Last name |
| `email` | Email address |
| `tel` | Full phone number |
| `username` | Username (login identifier) |
| `new-password` | New password (registration) |
| `current-password` | Existing password (login) |
| `one-time-code` | OTP/verification code |
| `street-address` | Full street address |
| `address-level1` | State/Province |
| `address-level2` | City |
| `postal-code` | ZIP/Postal code |
| `country-name` | Country name |
| `cc-name` | Cardholder name |
| `cc-number` | Credit card number |
| `cc-exp` | Card expiration (MM/YY) |
| `cc-csc` | Card security code |
| `bday` | Birthday |
| `organization` | Company name |

## Shipping vs Billing

```tsx
// ✅ Distinguish shipping and billing addresses
<fieldset>
  <legend>Shipping Address</legend>
  <input
    type="text"
    name="shippingAddress"
    autoComplete="shipping street-address"
  />
  <input
    type="text"
    name="shippingCity"
    autoComplete="shipping address-level2"
  />
</fieldset>

<fieldset>
  <legend>Billing Address</legend>
  <input
    type="text"
    name="billingAddress"
    autoComplete="billing street-address"
  />
  <input
    type="text"
    name="billingCity"
    autoComplete="billing address-level2"
  />
</fieldset>
```

## When to Disable Autocomplete

```tsx
// ✅ Legitimate cases to disable autocomplete
<input
  type="text"
  name="search"
  autoComplete="off"  // Search boxes
/>

<input
  type="text"
  name="captcha"
  autoComplete="off"  // CAPTCHA responses
/>

// ❌ Don't disable for these (user hostile)
<input type="email" autoComplete="off" />    // Email
<input type="password" autoComplete="off" /> // Password
<input type="text" name="address" autoComplete="off" /> // Address
```

## Benefits

- Faster form completion
- Fewer input errors
- Better mobile experience (native keyboards)
- Password manager integration
- WCAG 1.3.5 compliance

---

---
title: Display Form Errors Clearly
impact: HIGH
impactDescription: WCAG 2.1 Level A - Error identification
tags: forms, errors, validation, accessibility, ux
---

## Display Form Errors Clearly

**Impact: HIGH (WCAG 2.1 Level A - Error identification)**

Display form errors clearly and consistently to help users identify and correct mistakes. Error messages should be visible, associated with their fields, and provide actionable guidance.

## Bad Example

```html
<!-- Anti-pattern: Poor error display -->
<form>
  <!-- Error message far from input -->
  <div class="errors">
    <p>Email is invalid</p>
    <p>Password is required</p>
  </div>

  <input type="text" name="email" value="bad-email">
  <input type="password" name="password">

  <!-- Generic JavaScript alert -->
  <script>
    function validate() {
      alert('There are errors in your form');
    }
  </script>

  <!-- Error shown only by color change -->
  <input type="text" style="border-color: red;" placeholder="Enter name">

  <!-- Tooltip-only errors (disappear quickly) -->
  <input type="email" title="Please enter a valid email">

  <!-- Clearing the field on error -->
  <input type="text" id="phone" oninvalid="this.value=''">
</form>
```

## Good Example

```html
<!-- Correct approach: Clear, accessible error display -->
<form id="contact-form" novalidate>
  <!-- Error summary at top of form -->
  <div id="error-summary"
       role="alert"
       aria-live="polite"
       class="error-summary"
       hidden>
    <h2>Please correct the following errors:</h2>
    <ul id="error-list">
      <!-- Populated by JavaScript -->
    </ul>
  </div>

  <div class="form-group" id="email-group">
    <label for="email">
      Email Address
      <span class="required" aria-hidden="true">*</span>
    </label>
    <input type="email"
           id="email"
           name="email"
           required
           aria-required="true"
           aria-describedby="email-hint email-error"
           aria-invalid="false">
    <p id="email-hint" class="hint">Enter your work email address</p>
    <p id="email-error" class="field-error" hidden>
      <span class="error-icon" aria-hidden="true">
        <svg width="16" height="16" viewBox="0 0 16 16">
          <circle cx="8" cy="8" r="7" fill="none" stroke="currentColor" stroke-width="2"/>
          <path d="M8 4v5M8 11v1" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
        </svg>
      </span>
      <span class="error-text"></span>
    </p>
  </div>

  <div class="form-group" id="phone-group">
    <label for="phone">Phone Number</label>
    <input type="tel"
           id="phone"
           name="phone"
           aria-describedby="phone-error"
           aria-invalid="false">
    <p id="phone-error" class="field-error" hidden>
      <span class="error-icon" aria-hidden="true">
        <svg width="16" height="16" viewBox="0 0 16 16">
          <circle cx="8" cy="8" r="7" fill="none" stroke="currentColor" stroke-width="2"/>
          <path d="M8 4v5M8 11v1" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
        </svg>
      </span>
      <span class="error-text"></span>
    </p>
  </div>

  <div class="form-group" id="message-group">
    <label for="message">
      Message
      <span class="required" aria-hidden="true">*</span>
    </label>
    <textarea id="message"
              name="message"
              required
              aria-required="true"
              aria-describedby="message-counter message-error"
              maxlength="500"
              rows="4"></textarea>
    <div class="field-footer">
      <p id="message-counter" class="character-counter">0/500 characters</p>
      <p id="message-error" class="field-error" hidden>
        <span class="error-icon" aria-hidden="true">
          <svg width="16" height="16" viewBox="0 0 16 16">
            <circle cx="8" cy="8" r="7" fill="none" stroke="currentColor" stroke-width="2"/>
            <path d="M8 4v5M8 11v1" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
          </svg>
        </span>
        <span class="error-text"></span>
      </p>
    </div>
  </div>

  <button type="submit">Send Message</button>
</form>

<script>
const form = document.getElementById('contact-form');
const errorSummary = document.getElementById('error-summary');
const errorList = document.getElementById('error-list');

form.addEventListener('submit', function(e) {
  e.preventDefault();
  clearAllErrors();

  const errors = validateForm();

  if (errors.length > 0) {
    displayErrors(errors);
  } else {
    submitForm();
  }
});

function validateForm() {
  const errors = [];

  // Email validation
  const email = document.getElementById('email');
  if (!email.value) {
    errors.push({ field: 'email', message: 'Email address is required' });
  } else if (!isValidEmail(email.value)) {
    errors.push({ field: 'email', message: 'Enter a valid email address, like name@company.com' });
  }

  // Phone validation (optional but must be valid if provided)
  const phone = document.getElementById('phone');
  if (phone.value && !isValidPhone(phone.value)) {
    errors.push({ field: 'phone', message: 'Enter a valid phone number, like (555) 123-4567' });
  }

  // Message validation
  const message = document.getElementById('message');
  if (!message.value.trim()) {
    errors.push({ field: 'message', message: 'Message is required' });
  } else if (message.value.length < 10) {
    errors.push({ field: 'message', message: 'Message must be at least 10 characters' });
  }

  return errors;
}

function displayErrors(errors) {
  // Build error summary
  errorList.innerHTML = errors.map(error =>
    `<li><a href="#${error.field}">${error.message}</a></li>`
  ).join('');

  errorSummary.hidden = false;

  // Display inline errors
  errors.forEach(error => {
    const input = document.getElementById(error.field);
    const errorEl = document.getElementById(`${error.field}-error`);
    const errorText = errorEl.querySelector('.error-text');

    input.setAttribute('aria-invalid', 'true');
    errorText.textContent = error.message;
    errorEl.hidden = false;

    // Add error class to form group
    document.getElementById(`${error.field}-group`).classList.add('has-error');
  });

  // Focus error summary (or first error field)
  errorSummary.scrollIntoView({ behavior: 'smooth', block: 'start' });
  errorList.querySelector('a').focus();
}

function clearAllErrors() {
  errorSummary.hidden = true;
  errorList.innerHTML = '';

  document.querySelectorAll('.field-error').forEach(el => {
    el.hidden = true;
  });

  document.querySelectorAll('[aria-invalid="true"]').forEach(el => {
    el.setAttribute('aria-invalid', 'false');
  });

  document.querySelectorAll('.has-error').forEach(el => {
    el.classList.remove('has-error');
  });
}
</script>

<style>
.error-summary {
  background: #fef2f2;
  border: 2px solid #dc2626;
  border-radius: 8px;
  padding: 1rem;
  margin-bottom: 1.5rem;
}

.error-summary h2 {
  color: #991b1b;
  font-size: 1rem;
  margin: 0 0 0.5rem;
}

.error-summary ul {
  margin: 0;
  padding-left: 1.5rem;
}

.error-summary a {
  color: #dc2626;
}

.field-error {
  color: #dc2626;
  font-size: 0.875rem;
  margin-top: 0.5rem;
  display: flex;
  align-items: flex-start;
  gap: 0.5rem;
}

.has-error input,
.has-error textarea {
  border-color: #dc2626;
  border-width: 2px;
}

.has-error input:focus,
.has-error textarea:focus {
  outline-color: #dc2626;
  box-shadow: 0 0 0 3px rgba(220, 38, 38, 0.15);
}
</style>
```

## Why

Proper error display is crucial for usability and accessibility:

1. **Discoverability**: Users must be able to find and understand errors quickly.

2. **Association**: Errors must be clearly linked to their corresponding fields.

3. **Screen Readers**: Programmatic association ensures errors are announced.

4. **Persistence**: Errors should remain visible until corrected.

5. **Guidance**: Messages should explain how to fix the problem.

Error display best practices:

1. **Inline errors**: Place error messages directly below or beside their fields
2. **Error summary**: For forms with multiple errors, list all errors at the top
3. **Visual indicators**: Use icons, borders, and background colors (not just color)
4. **Persistent display**: Keep errors visible until the user corrects them
5. **Clear language**: Tell users what went wrong and how to fix it
6. **Focus management**: Move focus to error summary or first error field
7. **ARIA attributes**: Use `aria-invalid`, `aria-describedby`, and live regions
8. **Don't clear valid data**: Preserve correctly entered information
9. **Real-time feedback**: Clear errors as soon as user corrects them
10. **Consistent positioning**: Always show errors in the same location relative to inputs

---

---
title: Implement Smart Inline Validation
impact: MEDIUM
impactDescription: Reduces errors by 20-40%
tags: forms, validation, ux, inline-validation
---

## Implement Smart Inline Validation

**Impact: MEDIUM (Reduces errors by 20-40%)**

Implement inline validation that provides immediate, contextual feedback as users complete form fields. Validate at the right moment to help without interrupting.

## Bad Example

```html
<!-- Anti-pattern: Aggressive inline validation -->
<form>
  <input type="email"
         id="email"
         oninput="validateEmail()"
         placeholder="Email">
  <span id="email-error"></span>
</form>

<script>
// Validates on every keystroke (too aggressive)
function validateEmail() {
  const email = document.getElementById('email').value;
  const error = document.getElementById('email-error');

  // Shows error immediately when user starts typing
  if (!email.includes('@')) {
    error.textContent = 'Invalid email!';
    error.style.color = 'red';
  } else {
    error.textContent = '';
  }
}
</script>

<!-- Anti-pattern: No inline validation (only on submit) -->
<form onsubmit="return validate()">
  <input type="text" name="username">
  <input type="password" name="password">
  <!-- User only finds out about errors after clicking submit -->
  <button type="submit">Sign Up</button>
</form>
```

## Good Example

```html
<!-- Correct approach: Balanced inline validation -->
<form id="signup-form" novalidate>
  <!-- Email with validation on blur and after correction -->
  <div class="form-group" data-validate="email">
    <label for="email">Email Address *</label>
    <input type="email"
           id="email"
           name="email"
           required
           aria-required="true"
           aria-describedby="email-hint email-feedback"
           autocomplete="email">
    <p id="email-hint" class="hint">We'll send your confirmation here</p>
    <div id="email-feedback"
         class="feedback"
         role="status"
         aria-live="polite"></div>
  </div>

  <!-- Username with async validation -->
  <div class="form-group" data-validate="username">
    <label for="username">Username *</label>
    <input type="text"
           id="username"
           name="username"
           required
           aria-required="true"
           aria-describedby="username-hint username-feedback"
           pattern="^[a-zA-Z0-9_]{3,20}$"
           autocomplete="username">
    <p id="username-hint" class="hint">3-20 characters, letters, numbers, underscores</p>
    <div id="username-feedback"
         class="feedback"
         role="status"
         aria-live="polite"></div>
  </div>

  <!-- Password with real-time strength indicator -->
  <div class="form-group" data-validate="password">
    <label for="password">Password *</label>
    <input type="password"
           id="password"
           name="password"
           required
           aria-required="true"
           aria-describedby="password-requirements password-feedback"
           minlength="8"
           autocomplete="new-password">

    <div id="password-requirements" class="requirements">
      <span class="requirements-label">Requirements:</span>
      <ul class="requirements-list">
        <li data-requirement="length">
          <span class="req-icon" aria-hidden="true"></span>
          <span class="req-text">At least 8 characters</span>
        </li>
        <li data-requirement="uppercase">
          <span class="req-icon" aria-hidden="true"></span>
          <span class="req-text">One uppercase letter</span>
        </li>
        <li data-requirement="lowercase">
          <span class="req-icon" aria-hidden="true"></span>
          <span class="req-text">One lowercase letter</span>
        </li>
        <li data-requirement="number">
          <span class="req-icon" aria-hidden="true"></span>
          <span class="req-text">One number</span>
        </li>
      </ul>
    </div>

    <div class="password-strength">
      <div class="strength-bar">
        <div class="strength-fill" id="strength-fill"></div>
      </div>
      <span id="strength-text" class="strength-text"></span>
    </div>

    <div id="password-feedback"
         class="feedback"
         role="status"
         aria-live="polite"></div>
  </div>

  <!-- Confirm password with match validation -->
  <div class="form-group" data-validate="confirmPassword">
    <label for="confirm-password">Confirm Password *</label>
    <input type="password"
           id="confirm-password"
           name="confirmPassword"
           required
           aria-required="true"
           aria-describedby="confirm-feedback"
           autocomplete="new-password">
    <div id="confirm-feedback"
         class="feedback"
         role="status"
         aria-live="polite"></div>
  </div>

  <button type="submit">Create Account</button>
</form>

<script>
// Validation timing configuration
const validationConfig = {
  email: { on: 'blur', revalidate: 'input' },
  username: { on: 'blur', revalidate: 'input', debounce: 500 },
  password: { on: 'input', immediate: true },
  confirmPassword: { on: 'blur', revalidate: 'input' }
};

// Track if field has been touched
const touchedFields = new Set();

// Debounce utility
function debounce(fn, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn.apply(this, args), delay);
  };
}

// Email validation
const emailInput = document.getElementById('email');
emailInput.addEventListener('blur', () => {
  touchedFields.add('email');
  validateEmail();
});
emailInput.addEventListener('input', () => {
  if (touchedFields.has('email')) {
    validateEmail();
  }
});

function validateEmail() {
  const email = emailInput.value.trim();
  const feedback = document.getElementById('email-feedback');

  if (!email) {
    showError(emailInput, feedback, 'Email address is required');
    return false;
  }

  if (!isValidEmail(email)) {
    showError(emailInput, feedback, 'Please enter a valid email (e.g., name@example.com)');
    return false;
  }

  showSuccess(emailInput, feedback, 'Email looks good');
  return true;
}

// Username validation with async availability check
const usernameInput = document.getElementById('username');
const checkUsername = debounce(validateUsername, 500);

usernameInput.addEventListener('blur', () => {
  touchedFields.add('username');
  validateUsername();
});
usernameInput.addEventListener('input', () => {
  if (touchedFields.has('username')) {
    checkUsername();
  }
});

async function validateUsername() {
  const username = usernameInput.value.trim();
  const feedback = document.getElementById('username-feedback');

  if (!username) {
    showError(usernameInput, feedback, 'Username is required');
    return false;
  }

  if (username.length < 3) {
    showError(usernameInput, feedback, 'Username must be at least 3 characters');
    return false;
  }

  if (!/^[a-zA-Z0-9_]+$/.test(username)) {
    showError(usernameInput, feedback, 'Only letters, numbers, and underscores allowed');
    return false;
  }

  // Check availability
  showPending(usernameInput, feedback, 'Checking availability...');

  try {
    const response = await fetch(`/api/check-username?username=${username}`);
    const { available } = await response.json();

    if (available) {
      showSuccess(usernameInput, feedback, 'Username is available');
      return true;
    } else {
      showError(usernameInput, feedback, 'Username is already taken');
      return false;
    }
  } catch (error) {
    showError(usernameInput, feedback, 'Could not check availability');
    return false;
  }
}

// Password validation with real-time feedback
const passwordInput = document.getElementById('password');
passwordInput.addEventListener('input', validatePassword);

function validatePassword() {
  const password = passwordInput.value;
  const requirements = {
    length: password.length >= 8,
    uppercase: /[A-Z]/.test(password),
    lowercase: /[a-z]/.test(password),
    number: /[0-9]/.test(password)
  };

  // Update requirement indicators
  Object.entries(requirements).forEach(([req, met]) => {
    const el = document.querySelector(`[data-requirement="${req}"]`);
    el.classList.toggle('met', met);
    el.classList.toggle('unmet', !met);
  });

  // Calculate and show strength
  const strength = Object.values(requirements).filter(Boolean).length;
  updateStrengthIndicator(strength);

  // Only show feedback after user has typed something
  if (password.length > 0) {
    touchedFields.add('password');
  }

  return Object.values(requirements).every(Boolean);
}

function updateStrengthIndicator(strength) {
  const fill = document.getElementById('strength-fill');
  const text = document.getElementById('strength-text');

  const levels = [
    { label: '', color: '', width: '0%' },
    { label: 'Weak', color: '#dc2626', width: '25%' },
    { label: 'Fair', color: '#f59e0b', width: '50%' },
    { label: 'Good', color: '#3b82f6', width: '75%' },
    { label: 'Strong', color: '#22c55e', width: '100%' }
  ];

  const level = levels[strength];
  fill.style.width = level.width;
  fill.style.backgroundColor = level.color;
  text.textContent = level.label;
  text.style.color = level.color;
}

// Confirm password
const confirmInput = document.getElementById('confirm-password');
confirmInput.addEventListener('blur', () => {
  touchedFields.add('confirmPassword');
  validateConfirm();
});
confirmInput.addEventListener('input', () => {
  if (touchedFields.has('confirmPassword')) {
    validateConfirm();
  }
});

function validateConfirm() {
  const confirm = confirmInput.value;
  const password = passwordInput.value;
  const feedback = document.getElementById('confirm-feedback');

  if (!confirm) {
    showError(confirmInput, feedback, 'Please confirm your password');
    return false;
  }

  if (confirm !== password) {
    showError(confirmInput, feedback, 'Passwords do not match');
    return false;
  }

  showSuccess(confirmInput, feedback, 'Passwords match');
  return true;
}

// Feedback display helpers
function showError(input, feedback, message) {
  input.setAttribute('aria-invalid', 'true');
  feedback.className = 'feedback error';
  feedback.innerHTML = `<span class="icon">✕</span> ${message}`;
}

function showSuccess(input, feedback, message) {
  input.setAttribute('aria-invalid', 'false');
  feedback.className = 'feedback success';
  feedback.innerHTML = `<span class="icon">✓</span> ${message}`;
}

function showPending(input, feedback, message) {
  input.removeAttribute('aria-invalid');
  feedback.className = 'feedback pending';
  feedback.innerHTML = `<span class="spinner"></span> ${message}`;
}
</script>

<style>
.feedback.error { color: #dc2626; }
.feedback.success { color: #22c55e; }
.feedback.pending { color: #6b7280; }
.requirements-list li.met { color: #22c55e; }
.requirements-list li.unmet { color: #6b7280; }
</style>
```

## Why

Inline validation improves form completion:

1. **Immediate Feedback**: Users know right away if data is valid.
2. **Error Prevention**: Catches mistakes before submission.
3. **Reduced Frustration**: No surprise errors at the end.
4. **Guided Input**: Real-time requirements help users succeed.

Validation timing guidelines:

| Field Type | Validate On | Revalidate On |
|------------|-------------|---------------|
| Email, URL | Blur | Input (after first blur) |
| Username | Blur + debounce | Input (after first blur) |
| Password | Input (immediate) | - |
| Confirm Password | Blur | Input (after first blur) |
| Required text | Blur | Input (after first blur) |

Key principles:

1. **Don't validate empty fields on input** - Wait for blur
2. **Revalidate on input after first error** - Clear errors as user fixes them
3. **Password is special** - Real-time feedback for requirements
4. **Debounce async validations** - Don't overwhelm server
5. **Show positive feedback** - Confirm when input is valid
6. **Preserve error until fixed** - Don't flash errors on/off

---

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

---

---
title: Design Effective Multi-Step Forms
impact: MEDIUM
impactDescription: Increases completion rates by 15-30%
tags: forms, multi-step, ux, progress-indication
---

## Design Effective Multi-Step Forms

**Impact: MEDIUM (Increases completion rates by 15-30%)**

Design multi-step forms that are easy to navigate, maintain context, and don't overwhelm users. Break complex forms into logical, manageable steps.

## Bad Example

```html
<!-- Anti-pattern: Poor multi-step form -->
<form>
  <!-- No progress indication -->
  <!-- No step numbers or titles -->

  <div class="step" id="step1">
    <input type="text" placeholder="Name">
    <input type="email" placeholder="Email">
    <button type="button" onclick="nextStep()">Next</button>
  </div>

  <div class="step" id="step2" hidden>
    <input type="text" placeholder="Address">
    <!-- No back button -->
    <button type="button" onclick="nextStep()">Next</button>
  </div>

  <div class="step" id="step3" hidden>
    <!-- No summary of previous steps -->
    <button type="submit">Submit</button>
  </div>
</form>

<script>
let step = 1;
function nextStep() {
  // Doesn't validate before proceeding
  document.getElementById(`step${step}`).hidden = true;
  step++;
  document.getElementById(`step${step}`).hidden = false;
  // Doesn't save progress
  // Doesn't announce step change to screen readers
}
</script>
```

## Good Example

```html
<!-- Correct approach: Accessible multi-step form -->
<div class="multi-step-form">
  <!-- Progress indicator -->
  <nav aria-label="Form progress">
    <ol class="progress-steps" role="list">
      <li class="step-indicator current"
          aria-current="step">
        <span class="step-number">1</span>
        <span class="step-label">Personal Info</span>
      </li>
      <li class="step-indicator">
        <span class="step-number">2</span>
        <span class="step-label">Address</span>
      </li>
      <li class="step-indicator">
        <span class="step-number">3</span>
        <span class="step-label">Review</span>
      </li>
    </ol>
    <div class="progress-bar">
      <div class="progress-fill" style="width: 33%;"
           role="progressbar"
           aria-valuenow="1"
           aria-valuemin="1"
           aria-valuemax="3"
           aria-label="Step 1 of 3"></div>
    </div>
  </nav>

  <!-- Status announcement for screen readers -->
  <div id="step-status"
       role="status"
       aria-live="polite"
       class="visually-hidden">
  </div>

  <form id="registration-form" novalidate>
    <!-- Step 1: Personal Information -->
    <fieldset id="step-1" class="form-step">
      <legend>
        <span class="step-title">Step 1 of 3: Personal Information</span>
      </legend>

      <div class="form-group">
        <label for="full-name">Full Name *</label>
        <input type="text"
               id="full-name"
               name="fullName"
               required
               aria-required="true"
               autocomplete="name">
        <p id="name-error" class="error" hidden></p>
      </div>

      <div class="form-group">
        <label for="email">Email Address *</label>
        <input type="email"
               id="email"
               name="email"
               required
               aria-required="true"
               autocomplete="email">
        <p id="email-error" class="error" hidden></p>
      </div>

      <div class="form-group">
        <label for="phone">Phone Number</label>
        <input type="tel"
               id="phone"
               name="phone"
               autocomplete="tel">
      </div>

      <div class="form-actions">
        <button type="button"
                class="btn-next"
                onclick="goToStep(2)">
          Continue to Address
          <svg aria-hidden="true" class="arrow-icon"><!-- arrow right --></svg>
        </button>
      </div>
    </fieldset>

    <!-- Step 2: Address -->
    <fieldset id="step-2" class="form-step" hidden>
      <legend>
        <span class="step-title">Step 2 of 3: Address</span>
      </legend>

      <div class="form-group">
        <label for="street">Street Address *</label>
        <input type="text"
               id="street"
               name="street"
               required
               aria-required="true"
               autocomplete="street-address">
        <p id="street-error" class="error" hidden></p>
      </div>

      <div class="form-row">
        <div class="form-group">
          <label for="city">City *</label>
          <input type="text"
                 id="city"
                 name="city"
                 required
                 aria-required="true"
                 autocomplete="address-level2">
        </div>

        <div class="form-group">
          <label for="state">State *</label>
          <select id="state"
                  name="state"
                  required
                  aria-required="true"
                  autocomplete="address-level1">
            <option value="">Select...</option>
            <option value="CA">California</option>
            <option value="NY">New York</option>
            <!-- more states -->
          </select>
        </div>

        <div class="form-group">
          <label for="zip">ZIP Code *</label>
          <input type="text"
                 id="zip"
                 name="zip"
                 required
                 aria-required="true"
                 autocomplete="postal-code"
                 inputmode="numeric"
                 pattern="[0-9]{5}">
        </div>
      </div>

      <div class="form-actions">
        <button type="button"
                class="btn-back"
                onclick="goToStep(1)">
          <svg aria-hidden="true" class="arrow-icon"><!-- arrow left --></svg>
          Back
        </button>
        <button type="button"
                class="btn-next"
                onclick="goToStep(3)">
          Review Your Information
          <svg aria-hidden="true" class="arrow-icon"><!-- arrow right --></svg>
        </button>
      </div>
    </fieldset>

    <!-- Step 3: Review -->
    <fieldset id="step-3" class="form-step" hidden>
      <legend>
        <span class="step-title">Step 3 of 3: Review & Submit</span>
      </legend>

      <div class="review-section">
        <h3>Personal Information</h3>
        <dl class="review-list">
          <dt>Name</dt>
          <dd id="review-name"></dd>
          <dt>Email</dt>
          <dd id="review-email"></dd>
          <dt>Phone</dt>
          <dd id="review-phone"></dd>
        </dl>
        <button type="button"
                class="btn-edit"
                onclick="goToStep(1)">
          Edit Personal Info
        </button>
      </div>

      <div class="review-section">
        <h3>Address</h3>
        <dl class="review-list">
          <dt>Street</dt>
          <dd id="review-street"></dd>
          <dt>City, State ZIP</dt>
          <dd id="review-city-state-zip"></dd>
        </dl>
        <button type="button"
                class="btn-edit"
                onclick="goToStep(2)">
          Edit Address
        </button>
      </div>

      <div class="form-actions">
        <button type="button"
                class="btn-back"
                onclick="goToStep(2)">
          <svg aria-hidden="true" class="arrow-icon"><!-- arrow left --></svg>
          Back
        </button>
        <button type="submit" class="btn-submit">
          Submit Registration
        </button>
      </div>
    </fieldset>
  </form>
</div>

<script>
let currentStep = 1;
const totalSteps = 3;
const form = document.getElementById('registration-form');
const statusEl = document.getElementById('step-status');

function goToStep(newStep) {
  // Validate current step before proceeding forward
  if (newStep > currentStep && !validateStep(currentStep)) {
    return;
  }

  // Save current step data
  saveProgress();

  // Hide current step
  document.getElementById(`step-${currentStep}`).hidden = true;

  // Update progress indicators
  updateProgressIndicators(newStep);

  // Show new step
  currentStep = newStep;
  const newStepEl = document.getElementById(`step-${newStep}`);
  newStepEl.hidden = false;

  // If review step, populate summary
  if (newStep === 3) {
    populateReview();
  }

  // Focus management
  newStepEl.querySelector('legend').focus();

  // Announce step change
  const stepTitle = newStepEl.querySelector('.step-title').textContent;
  statusEl.textContent = `Now on ${stepTitle}`;

  // Scroll to top of form
  newStepEl.scrollIntoView({ behavior: 'smooth', block: 'start' });
}

function validateStep(step) {
  const stepEl = document.getElementById(`step-${step}`);
  const requiredFields = stepEl.querySelectorAll('[required]');
  let isValid = true;
  let firstError = null;

  requiredFields.forEach(field => {
    if (!field.validity.valid) {
      isValid = false;
      showFieldError(field);
      if (!firstError) firstError = field;
    } else {
      clearFieldError(field);
    }
  });

  if (firstError) {
    firstError.focus();
    statusEl.textContent = 'Please correct the errors before continuing.';
  }

  return isValid;
}

function updateProgressIndicators(newStep) {
  const indicators = document.querySelectorAll('.step-indicator');
  indicators.forEach((indicator, index) => {
    const stepNum = index + 1;
    indicator.classList.toggle('completed', stepNum < newStep);
    indicator.classList.toggle('current', stepNum === newStep);
    if (stepNum === newStep) {
      indicator.setAttribute('aria-current', 'step');
    } else {
      indicator.removeAttribute('aria-current');
    }
  });

  // Update progress bar
  const progressFill = document.querySelector('.progress-fill');
  const percentage = (newStep / totalSteps) * 100;
  progressFill.style.width = `${percentage}%`;
  progressFill.setAttribute('aria-valuenow', newStep);
}

function saveProgress() {
  const formData = new FormData(form);
  const data = Object.fromEntries(formData.entries());
  localStorage.setItem('registration-progress', JSON.stringify({
    step: currentStep,
    data: data
  }));
}

function populateReview() {
  document.getElementById('review-name').textContent =
    document.getElementById('full-name').value || 'Not provided';
  document.getElementById('review-email').textContent =
    document.getElementById('email').value;
  document.getElementById('review-phone').textContent =
    document.getElementById('phone').value || 'Not provided';
  document.getElementById('review-street').textContent =
    document.getElementById('street').value;
  document.getElementById('review-city-state-zip').textContent =
    `${document.getElementById('city').value}, ${document.getElementById('state').value} ${document.getElementById('zip').value}`;
}

// Restore progress on page load
window.addEventListener('load', () => {
  const saved = localStorage.getItem('registration-progress');
  if (saved) {
    const { step, data } = JSON.parse(saved);
    // Populate form fields
    Object.entries(data).forEach(([name, value]) => {
      const field = form.elements[name];
      if (field) field.value = value;
    });
    // Go to saved step
    if (step > 1) goToStep(step);
  }
});
</script>
```

## Why

Multi-step forms improve complex form UX:

1. **Reduced Cognitive Load**: Breaking forms into steps prevents overwhelming users.

2. **Progress Visibility**: Users know how much they've done and what's left.

3. **Data Preservation**: Users can go back without losing progress.

4. **Error Isolation**: Validation per step catches errors early.

5. **Completion Rates**: Shorter perceived forms have higher completion rates.

Multi-step form best practices:

1. **Clear progress**: Show current step and total steps
2. **Logical grouping**: Related fields together
3. **Back navigation**: Always allow returning to previous steps
4. **Data persistence**: Save progress (localStorage, server-side)
5. **Step validation**: Validate before advancing
6. **Review step**: Let users verify before submitting
7. **Edit capability**: Allow editing previous steps from review
8. **Focus management**: Focus first element of each step
9. **Screen reader announcements**: Announce step changes
10. **Save & continue later**: For long forms

---

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

---

---
title: Provide Clear Form Submission Feedback
impact: HIGH
impactDescription: Prevents user confusion and double submissions
tags: forms, feedback, ux, loading-states
---

## Provide Clear Form Submission Feedback

**Impact: HIGH (Prevents user confusion and double submissions)**

Provide clear, immediate feedback when users submit forms. Users should always know the current state: submitting, success, or failure.

## Bad Example

```html
<!-- Anti-pattern: No submission feedback -->
<form onsubmit="submitForm()">
  <input type="email" name="email" required>
  <button type="submit">Subscribe</button>
</form>

<script>
async function submitForm() {
  // No loading indication
  const response = await fetch('/api/subscribe', {
    method: 'POST',
    body: new FormData(form)
  });

  // Silent success - user doesn't know it worked
  if (response.ok) {
    console.log('Success');
  }

  // Unhelpful error
  if (!response.ok) {
    alert('Error');
  }
}
</script>
```

```html
<!-- Anti-pattern: Confusing states -->
<form>
  <input type="email" name="email">
  <!-- Button changes but no visual feedback -->
  <button type="submit" id="btn">Subscribe</button>
</form>

<script>
btn.textContent = 'Sending...';
// User can click multiple times
// No disabled state
// Page reloads on error, losing context
</script>
```

## Good Example

```html
<!-- Correct approach: Comprehensive submission feedback -->
<form id="newsletter-form" novalidate>
  <div class="form-group">
    <label for="email">Email Address</label>
    <input type="email"
           id="email"
           name="email"
           required
           aria-required="true"
           aria-describedby="email-error"
           autocomplete="email">
    <p id="email-error" class="error" hidden></p>
  </div>

  <button type="submit"
          id="submit-btn"
          aria-describedby="submit-status">
    <span class="btn-text">Subscribe</span>
    <span class="btn-loading" hidden>
      <svg class="spinner" aria-hidden="true" viewBox="0 0 24 24">
        <circle cx="12" cy="12" r="10" fill="none" stroke="currentColor" stroke-width="3"/>
      </svg>
      <span>Subscribing...</span>
    </span>
  </button>

  <!-- Status message for screen readers -->
  <div id="submit-status"
       role="status"
       aria-live="polite"
       class="visually-hidden">
  </div>
</form>

<!-- Success state (shown after successful submission) -->
<div id="success-message" class="success-panel" hidden>
  <svg class="success-icon" aria-hidden="true" viewBox="0 0 24 24">
    <path d="M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41z" fill="currentColor"/>
  </svg>
  <h2>You're subscribed!</h2>
  <p>Check your inbox for a confirmation email.</p>
  <button type="button" onclick="resetForm()">Subscribe another email</button>
</div>

<script>
const form = document.getElementById('newsletter-form');
const submitBtn = document.getElementById('submit-btn');
const btnText = submitBtn.querySelector('.btn-text');
const btnLoading = submitBtn.querySelector('.btn-loading');
const statusEl = document.getElementById('submit-status');
const successMessage = document.getElementById('success-message');

form.addEventListener('submit', async (e) => {
  e.preventDefault();

  // Validate
  const email = document.getElementById('email');
  if (!email.validity.valid) {
    showError(email, 'Please enter a valid email address');
    return;
  }

  // Set loading state
  setLoadingState(true);

  try {
    const response = await fetch('/api/subscribe', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email: email.value })
    });

    const data = await response.json();

    if (response.ok) {
      showSuccess();
    } else {
      // Server returned an error
      showError(email, data.message || 'Subscription failed. Please try again.');
      setLoadingState(false);
    }
  } catch (error) {
    // Network or other error
    showError(email, 'Network error. Please check your connection and try again.');
    setLoadingState(false);
  }
});

function setLoadingState(isLoading) {
  submitBtn.disabled = isLoading;
  submitBtn.setAttribute('aria-busy', isLoading);
  btnText.hidden = isLoading;
  btnLoading.hidden = !isLoading;

  // Announce to screen readers
  if (isLoading) {
    statusEl.textContent = 'Submitting your subscription...';
  }
}

function showSuccess() {
  form.hidden = true;
  successMessage.hidden = false;

  // Announce success
  statusEl.textContent = 'Success! You have been subscribed to our newsletter.';

  // Focus the success message for accessibility
  successMessage.focus();
}

function showError(input, message) {
  const errorEl = document.getElementById(`${input.id}-error`);
  input.setAttribute('aria-invalid', 'true');
  errorEl.textContent = message;
  errorEl.hidden = false;
  input.focus();

  // Announce error
  statusEl.textContent = `Error: ${message}`;
}

function resetForm() {
  form.reset();
  form.hidden = false;
  successMessage.hidden = true;
  document.getElementById('email').focus();
}
</script>

<style>
/* Loading spinner animation */
@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.spinner {
  width: 1rem;
  height: 1rem;
  animation: spin 1s linear infinite;
}

.spinner circle {
  stroke-dasharray: 50;
  stroke-dashoffset: 20;
}

/* Button states */
button[type="submit"] {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  min-width: 140px;
  justify-content: center;
}

button[type="submit"]:disabled {
  opacity: 0.7;
  cursor: wait;
}

/* Success panel */
.success-panel {
  text-align: center;
  padding: 2rem;
  background: #f0fdf4;
  border: 2px solid #22c55e;
  border-radius: 8px;
}

.success-icon {
  width: 48px;
  height: 48px;
  color: #22c55e;
}

/* Focus for success panel */
.success-panel:focus {
  outline: 3px solid #22c55e;
  outline-offset: 2px;
}
</style>
```

## Why

Submission feedback is critical for user confidence:

1. **Acknowledgment**: Users need to know their action was received.

2. **Prevent Duplicates**: Loading states prevent multiple submissions.

3. **Error Recovery**: Clear error messages help users fix issues.

4. **Accessibility**: Screen reader users need announcements for state changes.

5. **Trust**: Professional feedback builds confidence in your application.

Submission state best practices:

| State | Visual | Accessible | Technical |
|-------|--------|------------|-----------|
| Idle | Normal button | Standard label | Enabled |
| Loading | Spinner, disabled | aria-busy, announce | Prevent double-submit |
| Success | Confirmation UI | Announce, focus | Clear form or redirect |
| Error | Error message | Announce, focus | Keep form data |

Key implementation details:

1. **Disable during submission**: Prevent double-clicks
2. **Show loading indicator**: Spinner or progress bar
3. **Announce state changes**: Use aria-live regions
4. **Focus management**: Move focus to success/error message
5. **Preserve input**: Never clear form data on error
6. **Specific error messages**: Explain what went wrong
7. **Recovery path**: Provide clear next steps
8. **Timeout handling**: Handle slow or failed network requests
9. **Optimistic UI**: Consider showing success immediately for fast operations

---

---
title: Design User-Friendly Form Validation
impact: HIGH
impactDescription: Reduces form abandonment by 20-30%
tags: forms, validation, ux, error-handling
---

## Design User-Friendly Form Validation

**Impact: HIGH (Reduces form abandonment by 20-30%)**

Design form validation that helps users succeed. Good validation provides clear feedback, prevents frustration, and guides users to correct errors efficiently.

## Bad Example

```html
<!-- Anti-pattern: Poor validation UX -->
<form onsubmit="return validateForm()">
  <input type="text" id="email" placeholder="Email">
  <input type="text" id="password" placeholder="Password">
  <button type="submit">Sign Up</button>
</form>

<script>
function validateForm() {
  // Only validates on submit
  const email = document.getElementById('email').value;
  const password = document.getElementById('password').value;

  // Generic, unhelpful error message
  if (!email || !password) {
    alert('Please fill in all fields');
    return false;
  }

  // Regex-only validation with no feedback
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
    alert('Invalid email');
    return false;
  }

  // Unclear requirements
  if (password.length < 8) {
    alert('Password too short');
    return false;
  }

  return true;
}
</script>

<style>
/* Error state with no guidance */
.error { border-color: red; }
</style>
```

## Good Example

```html
<!-- Correct approach: User-friendly validation -->
<form id="signup-form" novalidate>
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
           aria-describedby="email-hint email-error"
           autocomplete="email">
    <p id="email-hint" class="hint">We'll send your confirmation here</p>
    <p id="email-error" class="error-message" role="alert" hidden></p>
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
           aria-describedby="password-requirements password-error"
           minlength="8"
           autocomplete="new-password">

    <!-- Show requirements upfront, not just on error -->
    <div id="password-requirements" class="requirements">
      <p>Password must contain:</p>
      <ul>
        <li id="req-length" class="requirement">
          <span class="icon" aria-hidden="true"></span>
          At least 8 characters
        </li>
        <li id="req-uppercase" class="requirement">
          <span class="icon" aria-hidden="true"></span>
          One uppercase letter
        </li>
        <li id="req-number" class="requirement">
          <span class="icon" aria-hidden="true"></span>
          One number
        </li>
      </ul>
    </div>
    <p id="password-error" class="error-message" role="alert" hidden></p>
  </div>

  <div class="form-group">
    <label for="confirm-password">
      Confirm Password
      <span class="required" aria-hidden="true">*</span>
    </label>
    <input type="password"
           id="confirm-password"
           name="confirmPassword"
           required
           aria-required="true"
           aria-describedby="confirm-error"
           autocomplete="new-password">
    <p id="confirm-error" class="error-message" role="alert" hidden></p>
  </div>

  <button type="submit" id="submit-btn">Create Account</button>
</form>

<script>
const form = document.getElementById('signup-form');
const emailInput = document.getElementById('email');
const passwordInput = document.getElementById('password');
const confirmInput = document.getElementById('confirm-password');

// Validate on blur (when leaving field)
emailInput.addEventListener('blur', validateEmail);
passwordInput.addEventListener('input', validatePassword);
confirmInput.addEventListener('blur', validateConfirmPassword);

// Also validate on submit
form.addEventListener('submit', function(e) {
  e.preventDefault();

  const isEmailValid = validateEmail();
  const isPasswordValid = validatePassword();
  const isConfirmValid = validateConfirmPassword();

  if (isEmailValid && isPasswordValid && isConfirmValid) {
    submitForm();
  } else {
    // Focus first invalid field
    const firstError = form.querySelector('[aria-invalid="true"]');
    if (firstError) firstError.focus();
  }
});

function validateEmail() {
  const email = emailInput.value.trim();
  const errorEl = document.getElementById('email-error');

  if (!email) {
    showError(emailInput, errorEl, 'Email address is required');
    return false;
  }

  if (!isValidEmail(email)) {
    showError(emailInput, errorEl, 'Please enter a valid email address (e.g., name@example.com)');
    return false;
  }

  clearError(emailInput, errorEl);
  return true;
}

function validatePassword() {
  const password = passwordInput.value;
  const requirements = {
    length: password.length >= 8,
    uppercase: /[A-Z]/.test(password),
    number: /[0-9]/.test(password)
  };

  // Update requirement indicators in real-time
  updateRequirement('req-length', requirements.length);
  updateRequirement('req-uppercase', requirements.uppercase);
  updateRequirement('req-number', requirements.number);

  const allMet = Object.values(requirements).every(Boolean);

  if (allMet) {
    passwordInput.setAttribute('aria-invalid', 'false');
    return true;
  }

  passwordInput.setAttribute('aria-invalid', 'true');
  return false;
}

function validateConfirmPassword() {
  const password = passwordInput.value;
  const confirm = confirmInput.value;
  const errorEl = document.getElementById('confirm-error');

  if (!confirm) {
    showError(confirmInput, errorEl, 'Please confirm your password');
    return false;
  }

  if (confirm !== password) {
    showError(confirmInput, errorEl, 'Passwords do not match');
    return false;
  }

  clearError(confirmInput, errorEl);
  return true;
}

function updateRequirement(id, met) {
  const el = document.getElementById(id);
  el.classList.toggle('met', met);
  el.classList.toggle('unmet', !met);
}

function showError(input, errorEl, message) {
  input.setAttribute('aria-invalid', 'true');
  errorEl.textContent = message;
  errorEl.hidden = false;
}

function clearError(input, errorEl) {
  input.setAttribute('aria-invalid', 'false');
  errorEl.hidden = true;
}

function isValidEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
</script>

<style>
.requirement.met .icon::before { content: '✓'; color: green; }
.requirement.unmet .icon::before { content: '○'; color: #666; }
.error-message { color: #dc2626; }
</style>
```

## Why

Good validation UX is critical because:

1. **Reduce Frustration**: Poor validation is one of the top causes of form abandonment.

2. **Guide Success**: Clear requirements help users enter correct data the first time.

3. **Instant Feedback**: Real-time validation prevents wasted submission attempts.

4. **Accessible**: Proper error association ensures all users understand issues.

5. **Trust Building**: Professional validation suggests a trustworthy service.

Validation timing strategies:

| Timing | Pros | Cons | Best For |
|--------|------|------|----------|
| On Submit | Non-intrusive | Delayed feedback | Simple forms |
| On Blur | Balanced feedback | Interrupts flow | Most fields |
| On Input | Immediate | Can be annoying | Password strength |
| Debounced | Prevents flashing | Slight delay | Async validation |

Best practices:

1. **Show requirements upfront** - Don't hide rules until users fail
2. **Validate at the right time** - Blur for most, input for passwords
3. **Be specific** - "Enter at least 8 characters" not "Invalid input"
4. **Suggest corrections** - "Did you mean gmail.com?"
5. **Allow submission attempts** - Don't disable submit, let users discover errors
6. **Focus first error** - Guide users to the problem
7. **Preserve valid input** - Never clear correctly entered data
8. **Format as user types** - Phone numbers, credit cards
9. **Use positive confirmation** - Checkmarks for met requirements

---

---
title: Respect Prefers-Reduced-Motion
impact: CRITICAL
impactDescription: WCAG 2.1 Level AA - Animation from interactions
tags: accessibility, animation, motion, vestibular
---

## Respect Prefers-Reduced-Motion

**Impact: CRITICAL (WCAG 2.1 Level AA - Animation from interactions)**

## Why It Matters

Some users experience motion sickness, vestibular disorders, or distraction from animations. The `prefers-reduced-motion` media query lets users opt out of non-essential motion. Respecting this preference is crucial for accessibility.

## Incorrect

```tsx
// ❌ No reduced motion consideration
<div className="animate-bounce">
  Bouncing element
</div>

// ❌ JavaScript animations without check
useEffect(() => {
  animate(element, { x: 100 }, { duration: 1000 })
}, [])
```

## Correct

### CSS Approach

```css
/* Base: no animation for reduced motion preference */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}

/* Or selectively disable */
.hero-animation {
  animation: float 3s ease-in-out infinite;
}

@media (prefers-reduced-motion: reduce) {
  .hero-animation {
    animation: none;
  }
}
```

### Tailwind CSS

```tsx
// ✅ Use motion-reduce variant
<div className="
  transition-transform duration-300
  hover:scale-105
  motion-reduce:transition-none
  motion-reduce:hover:transform-none
">
  Hover me
</div>

// ✅ Disable animation for reduced motion
<div className="
  animate-pulse
  motion-reduce:animate-none
">
  Loading...
</div>

// ✅ Simplify rather than remove
<div className="
  transition-all duration-300 ease-out
  motion-reduce:transition-opacity motion-reduce:duration-150
">
  {/* Keeps opacity fade but removes transform */}
</div>
```

### JavaScript Check

```tsx
// Hook for checking preference
function usePrefersReducedMotion(): boolean {
  const [prefersReducedMotion, setPrefersReducedMotion] = useState(false)

  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)')

    setPrefersReducedMotion(mediaQuery.matches)

    const handler = (event: MediaQueryListEvent) => {
      setPrefersReducedMotion(event.matches)
    }

    mediaQuery.addEventListener('change', handler)
    return () => mediaQuery.removeEventListener('change', handler)
  }, [])

  return prefersReducedMotion
}

// Usage
function AnimatedComponent() {
  const prefersReducedMotion = usePrefersReducedMotion()

  return (
    <motion.div
      initial={{ opacity: 0, y: prefersReducedMotion ? 0 : 20 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: prefersReducedMotion ? 0 : 0.3 }}
    >
      Content
    </motion.div>
  )
}
```

### Framer Motion Integration

```tsx
import { motion, useReducedMotion } from 'framer-motion'

function Card() {
  const shouldReduceMotion = useReducedMotion()

  return (
    <motion.div
      initial={{ opacity: 0, scale: shouldReduceMotion ? 1 : 0.9 }}
      animate={{ opacity: 1, scale: 1 }}
      transition={{
        duration: shouldReduceMotion ? 0.001 : 0.3,
      }}
      whileHover={shouldReduceMotion ? {} : { scale: 1.05 }}
    >
      Card content
    </motion.div>
  )
}
```

### Animation with Fallback

```tsx
// Show static alternative for reduced motion
function LoadingIndicator() {
  const prefersReducedMotion = usePrefersReducedMotion()

  if (prefersReducedMotion) {
    return <span>Loading...</span>  // Text instead of spinner
  }

  return (
    <div className="animate-spin rounded-full h-8 w-8 border-2 border-blue-600 border-t-transparent" />
  )
}
```

## What to Keep vs Remove

### Should Reduce/Remove
- Parallax effects
- Hover animations on non-interactive elements
- Loading spinners (use text)
- Page transition animations
- Auto-playing carousels
- Continuous animations (pulse, bounce)

### Can Keep (Essential Motion)
- Progress indicators
- Micro-interactions that provide feedback
- Opacity transitions (usually fine)
- Focus indicators
- State change indicators

```tsx
// ✅ Essential feedback - keep but simplify
<button
  className={cn(
    'transition-colors',  // Color change is usually OK
    'motion-reduce:transition-none',
    'motion-safe:hover:scale-105 motion-safe:active:scale-95'  // Scale only for motion-safe
  )}
>
  Submit
</button>
```

## Testing

```tsx
// Simulate in DevTools:
// Chrome: Rendering panel > Emulate CSS media feature > prefers-reduced-motion
// Firefox: about:config > ui.prefersReducedMotion (0=no-preference, 1=reduce)

// Or in CSS:
@media (prefers-reduced-motion: no-preference) {
  /* Animations for users who haven't opted out */
}

@media (prefers-reduced-motion: reduce) {
  /* Reduced/no animations */
}
```

## Benefits

- Accessibility for vestibular disorders
- Better experience for motion-sensitive users
- Respects user preferences
- WCAG 2.3.3 compliance
- Reduces CPU/battery usage

---

