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
