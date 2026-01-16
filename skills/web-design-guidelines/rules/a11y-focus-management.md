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
