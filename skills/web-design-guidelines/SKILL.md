---
name: web-design-guidelines
description: UI/UX best practices and accessibility guidelines. Use when reviewing UI code, checking accessibility, auditing forms, or ensuring web interface best practices. Triggers on "review UI", "check accessibility", "audit design", "review UX", or "check best practices".
license: MIT
metadata:
  author: agent-skills
  version: "1.0.0"
---

# Web Design Guidelines

Comprehensive UI/UX and accessibility guidelines for building inclusive, performant web interfaces. Contains 50+ rules covering accessibility, forms, performance, and user experience.

## When to Apply

Reference these guidelines when:
- Reviewing UI code for accessibility
- Implementing forms and interactions
- Optimizing performance
- Ensuring cross-browser compatibility
- Improving user experience

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Accessibility | CRITICAL | `a11y-` |
| 2 | Forms & Validation | CRITICAL | `form-` |
| 3 | Performance | HIGH | `perf-` |
| 4 | Animation & Motion | HIGH | `motion-` |
| 5 | Typography | MEDIUM | `typo-` |
| 6 | Touch & Interaction | MEDIUM | `touch-` |
| 7 | Internationalization | LOW | `i18n-` |

## Quick Reference

### 1. Accessibility (CRITICAL)

- `a11y-semantic-html` - Use semantic HTML elements
- `a11y-aria-labels` - Add ARIA labels to interactive elements
- `a11y-keyboard-nav` - Ensure keyboard navigation
- `a11y-focus-visible` - Visible focus indicators
- `a11y-color-contrast` - Sufficient color contrast
- `a11y-alt-text` - Meaningful image alt text
- `a11y-heading-hierarchy` - Proper heading structure
- `a11y-form-labels` - Associate labels with inputs
- `a11y-skip-links` - Skip navigation links
- `a11y-live-regions` - Announce dynamic content

### 2. Forms & Validation (CRITICAL)

- `form-autocomplete` - Use autocomplete attributes
- `form-input-types` - Correct input types
- `form-error-messages` - Clear error messages
- `form-validation-timing` - Validate at right time
- `form-required-fields` - Mark required fields
- `form-success-feedback` - Confirm successful actions
- `form-disabled-states` - Clear disabled states

### 3. Performance (HIGH)

- `perf-image-dimensions` - Set image dimensions
- `perf-lazy-loading` - Lazy load below-fold content
- `perf-virtualization` - Virtualize long lists
- `perf-preconnect` - Preconnect to required origins
- `perf-font-loading` - Optimize font loading
- `perf-layout-shifts` - Prevent layout shifts

### 4. Animation & Motion (HIGH)

- `motion-reduced` - Respect prefers-reduced-motion
- `motion-performance` - Use GPU-friendly animations
- `motion-meaningful` - Animations should have purpose
- `motion-duration` - Appropriate animation duration

### 5. Typography (MEDIUM)

- `typo-readable` - Readable font sizes
- `typo-line-height` - Appropriate line height
- `typo-measure` - Optimal line length
- `typo-hierarchy` - Clear visual hierarchy

### 6. Touch & Interaction (MEDIUM)

- `touch-targets` - Minimum touch target sizes
- `touch-tap-highlight` - Handle tap highlight
- `touch-scrolling` - Smooth scrolling behavior

### 7. Internationalization (LOW)

- `i18n-date-format` - Use Intl.DateTimeFormat
- `i18n-number-format` - Use Intl.NumberFormat
- `i18n-rtl-support` - Support RTL languages

## Essential Guidelines

### Semantic HTML

```tsx
// ❌ Div soup - no semantic meaning
<div className="header">
  <div className="nav">
    <div onClick={handleClick}>Home</div>
  </div>
</div>
<div className="content">
  <div className="title">Page Title</div>
  <div className="text">Content here...</div>
</div>

// ✅ Semantic HTML - accessible and meaningful
<header>
  <nav aria-label="Main navigation">
    <a href="/">Home</a>
  </nav>
</header>
<main>
  <article>
    <h1>Page Title</h1>
    <p>Content here...</p>
  </article>
</main>
```

### ARIA Labels

```tsx
// Interactive elements need accessible names
<button aria-label="Close dialog">
  <XIcon />
</button>

<button aria-label="Add to cart">
  <PlusIcon />
</button>

// Icon-only links
<a href="/settings" aria-label="Settings">
  <SettingsIcon />
</a>

// Decorative icons should be hidden
<span aria-hidden="true">🎉</span>
```

### Keyboard Navigation

```tsx
// All interactive elements must be keyboard accessible
function Dialog({ isOpen, onClose, children }) {
  // Trap focus inside dialog
  const dialogRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    if (isOpen) {
      dialogRef.current?.focus()
    }
  }, [isOpen])

  // Handle Escape key
  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'Escape') {
      onClose()
    }
  }

  return (
    <div
      ref={dialogRef}
      role="dialog"
      aria-modal="true"
      tabIndex={-1}
      onKeyDown={handleKeyDown}
    >
      {children}
      <button onClick={onClose}>Close</button>
    </div>
  )
}
```

### Focus Styles

```tsx
// ✅ Always visible focus styles
<button className="
  focus:outline-none
  focus-visible:ring-2
  focus-visible:ring-blue-500
  focus-visible:ring-offset-2
">
  Button
</button>

// ❌ Never remove focus outlines without replacement
<button className="outline-none focus:outline-none">
  Inaccessible
</button>
```

### Form Accessibility

```tsx
// ✅ Properly labeled form
<form>
  <div>
    <label htmlFor="email">
      Email address
      <span aria-hidden="true" className="text-red-500">*</span>
    </label>
    <input
      id="email"
      type="email"
      name="email"
      required
      aria-required="true"
      aria-describedby="email-hint email-error"
      autoComplete="email"
    />
    <p id="email-hint" className="text-gray-500 text-sm">
      We'll never share your email.
    </p>
    {error && (
      <p id="email-error" role="alert" className="text-red-500 text-sm">
        {error}
      </p>
    )}
  </div>

  <button type="submit">Subscribe</button>
</form>
```

### Respect Reduced Motion

```tsx
// CSS
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

// Tailwind
<div className="
  transition-transform duration-300
  hover:scale-105
  motion-reduce:transition-none
  motion-reduce:hover:transform-none
">
  Card
</div>

// JavaScript
const prefersReducedMotion = window.matchMedia(
  '(prefers-reduced-motion: reduce)'
).matches

function animate() {
  if (prefersReducedMotion) {
    // Skip or simplify animation
    return
  }
  // Full animation
}
```

### Image Handling

```tsx
// ✅ Proper image implementation
<img
  src="/hero.webp"
  alt="Team collaborating around a whiteboard"
  width={1200}
  height={600}
  loading="lazy"
  decoding="async"
/>

// ✅ Decorative images
<img src="/pattern.svg" alt="" aria-hidden="true" />

// ✅ Responsive images
<picture>
  <source
    srcSet="/hero-mobile.webp"
    media="(max-width: 768px)"
    type="image/webp"
  />
  <source srcSet="/hero.webp" type="image/webp" />
  <img src="/hero.jpg" alt="Hero description" width={1200} height={600} />
</picture>
```

### Touch Targets

```tsx
// ✅ Minimum 44x44px touch targets
<button className="min-h-[44px] min-w-[44px] p-2">
  <Icon className="w-6 h-6" />
</button>

// ✅ Adequate spacing between touch targets
<nav className="flex gap-2">
  <a href="/" className="p-3">Home</a>
  <a href="/about" className="p-3">About</a>
</nav>
```

### Color Contrast

```css
/* WCAG AA requires 4.5:1 for normal text, 3:1 for large text */

/* ❌ Insufficient contrast */
.low-contrast {
  color: #999999;        /* Gray on white: ~2.8:1 */
  background: white;
}

/* ✅ Sufficient contrast */
.good-contrast {
  color: #595959;        /* Darker gray: ~7:1 */
  background: white;
}

/* ✅ Don't rely on color alone */
.error {
  color: #dc2626;
  border-left: 4px solid #dc2626;  /* Visual indicator */
}
```

### Live Regions

```tsx
// Announce dynamic content to screen readers
<div
  role="status"
  aria-live="polite"
  aria-atomic="true"
  className="sr-only"
>
  {message}
</div>

// Toast notifications
function Toast({ message }: { message: string }) {
  return (
    <div
      role="alert"
      aria-live="assertive"
      className="fixed bottom-4 right-4 bg-gray-900 text-white p-4 rounded"
    >
      {message}
    </div>
  )
}
```

## Output Format

When auditing code, output findings in this format:

```
file:line - [category] Description of issue
```

Example:
```
src/components/Button.tsx:15 - [a11y] Missing aria-label on icon-only button
src/pages/Home.tsx:42 - [perf] Image missing width/height attributes
src/components/Form.tsx:28 - [form] Input missing associated label
```

## How to Use

Read individual rule files for detailed explanations:

```
rules/a11y-semantic-html.md
rules/form-autocomplete.md
rules/motion-reduced.md
```
