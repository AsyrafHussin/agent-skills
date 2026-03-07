---
title: "V4 Custom Utilities and Variants"
prefix: v4-custom-utilities
impact: HIGH
tags: [tailwind, v4, custom-utilities, plugins, variants, css]
version: v4 only
---

## Why It Matters

Tailwind v4 replaces JavaScript plugins with CSS-native directives: `@utility` for custom utility classes and `@custom-variant` for custom variants. No `plugin()` API, no JavaScript — everything lives in your CSS file.

## Incorrect

```js
// ❌ v3 plugin API — does not work in v4
const plugin = require('tailwindcss/plugin')

module.exports = {
  plugins: [
    plugin(function ({ addUtilities, addVariant }) {
      addUtilities({
        '.scrollbar-hide': { '-ms-overflow-style': 'none', 'scrollbar-width': 'none' },
      })
      addVariant('hocus', ['&:hover', '&:focus'])
    }),
  ],
}
```

## Correct

### Custom utilities with @utility

```css
@import "tailwindcss";

/* Simple custom utility */
@utility scrollbar-hide {
  -ms-overflow-style: none;
  scrollbar-width: none;

  &::-webkit-scrollbar {
    display: none;
  }
}

/* Utility with variant support */
@utility text-balance {
  text-wrap: balance;
}
```

```html
<!-- Use like any built-in utility -->
<div class="scrollbar-hide overflow-y-auto">
<p class="text-balance">
```

### Custom variants with @custom-variant

```css
@import "tailwindcss";

/* Combine hover + focus into one variant */
@custom-variant hocus (&:hover, &:focus);

/* Target elements inside a specific parent */
@custom-variant sidebar-open (.sidebar-open &);

/* Dark mode scoped to a class (if you need custom selector) */
@custom-variant dark (&:where(.dark, .dark *));

/* RTL support */
@custom-variant rtl ([dir="rtl"] &);
```

```html
<!-- hocus: applies on both hover and focus -->
<button class="hocus:bg-blue-600 hocus:text-white">

<!-- sidebar-open: applies when ancestor has .sidebar-open -->
<nav class="sidebar-open:translate-x-0 -translate-x-full">
```

### Using official plugins in v4

```bash
npm install @tailwindcss/typography @tailwindcss/forms
```

```css
/* app.css — import plugins directly in CSS */
@import "tailwindcss";
@plugin "@tailwindcss/typography";
@plugin "@tailwindcss/forms";
```

## Recommended Patterns

| v3 (JS plugin API) | v4 (CSS directive) |
|--------------------|--------------------|
| `addUtilities({ '.foo': {...} })` | `@utility foo { ... }` |
| `addVariant('bar', '...')` | `@custom-variant bar (...)` |
| `require('@tailwindcss/forms')` | `@plugin "@tailwindcss/forms"` |
| `require('@tailwindcss/typography')` | `@plugin "@tailwindcss/typography"` |

Reference: https://tailwindcss.com/docs/adding-custom-styles
