# Web Design Guidelines

UI/UX best practices and accessibility guidelines for web interfaces.

## Overview

This skill provides guidance for:
- Accessibility compliance (WCAG)
- Form usability and validation
- Performance optimization
- Animation and motion
- Touch interactions
- Internationalization

## Categories

### 1. Accessibility (Critical)
Semantic HTML, ARIA, keyboard navigation, focus management, and color contrast.

### 2. Forms & Validation (Critical)
Input types, autocomplete, error handling, and validation timing.

### 3. Performance (High)
Images, lazy loading, virtualization, and layout shifts.

### 4. Animation & Motion (High)
Reduced motion, GPU-friendly animations, and meaningful transitions.

### 5. Typography (Medium)
Readability, line height, and visual hierarchy.

### 6. Touch & Interaction (Medium)
Touch targets, tap feedback, and scrolling.

### 7. Internationalization (Low)
Date/number formatting and RTL support.

## Usage

Ask Claude to:
- "Review my UI for accessibility"
- "Check this form for best practices"
- "Audit my component for UX issues"
- "Check accessibility on this page"

## Output Format

Issues are reported as:
```
file:line - [category] Description
```

## Key Guidelines

### Accessibility
- Use semantic HTML elements
- Provide ARIA labels for interactive elements
- Ensure keyboard navigation works
- Maintain visible focus indicators
- Meet color contrast requirements

### Forms
- Use correct input types
- Add autocomplete attributes
- Provide clear error messages
- Associate labels with inputs

### Performance
- Set image dimensions
- Lazy load below-fold content
- Prevent layout shifts

### Motion
- Respect prefers-reduced-motion
- Use GPU-friendly transforms
- Keep animations purposeful

## References

- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [MDN Accessibility](https://developer.mozilla.org/en-US/docs/Web/Accessibility)
- [Web.dev Accessibility](https://web.dev/accessibility/)
