# Rule Sections

## Priority Levels

| Level | Description | When to Apply |
|-------|-------------|---------------|
| CRITICAL | Essential for any Tailwind project | Always |
| HIGH | Important patterns for maintainable code | Most projects |
| MEDIUM | Advanced patterns and optimizations | When scaling |
| LOW | Edge cases and specialized needs | Specific use cases |

## Section Overview

### Responsive Design (CRITICAL)
Rules for mobile-first responsive design patterns. These are fundamental to creating adaptive layouts that work across all device sizes. Includes breakpoint usage, container queries, fluid typography, and responsive grid systems.

**Focus:** Mobile-first approach, breakpoint ordering, fluid scaling
**Impact:** User experience on all devices, accessibility, performance

### Dark Mode (CRITICAL)
Rules for implementing dark mode support. Modern applications must support both light and dark themes seamlessly. Covers setup strategies, color schemes, transitions, and custom color palettes for dual-theme support.

**Focus:** Theme strategies, color management, smooth transitions
**Impact:** User preference support, reduced eye strain, modern UX

### Component Patterns (HIGH)
Rules for building reusable, consistent components. Essential patterns for buttons, forms, cards, tables, modals, and navigation. Includes conditional class handling with clsx/cn utility for clean, maintainable component code.

**Focus:** Reusable patterns, consistent styling, component composition
**Impact:** Code maintainability, design consistency, development speed

### Custom Configuration — v3 only (HIGH)
Rules for extending Tailwind's v3 theme via `tailwind.config.js`. Covers extending vs overriding, custom colors with proper scales, font families with fallbacks, custom spacing values, plugin usage, and preset sharing across projects. **For v4 projects, use `@theme {}` instead — see V4 & Migration.**

**Focus:** Theme extension, design system integration, configuration patterns
**Impact:** Design system alignment, team consistency, flexibility

### V4 & Migration (HIGH)
Rules for Tailwind CSS v4 — a CSS-first rewrite that replaces `tailwind.config.js` with `@theme {}` in CSS. Covers installation with the Vite/PostCSS plugin, theme configuration with `@theme`, custom utilities with `@utility` and `@custom-variant`, and the full step-by-step migration path from v3 to v4.

**Focus:** CSS-first configuration, v4 setup, v3→v4 migration path
**Impact:** Modern tooling, zero-JS config, access to new v4 features
**Rules:** `v4-installation`, `v4-theme-configuration`, `v4-custom-utilities`, `v4-migration`

### Spacing & Typography (MEDIUM)
Rules for consistent spacing and typography systems. Includes spacing scale usage, margin/padding patterns, typography hierarchy, line height management, and responsive text sizing.

**Focus:** Vertical rhythm, typography scale, consistent spacing
**Impact:** Visual consistency, readability, design system cohesion

### Animation (MEDIUM)
Rules for adding animations and transitions. Covers transition utilities, custom keyframes, motion preferences (respecting reduced motion), and performance considerations for animations.

**Focus:** Smooth transitions, custom animations, accessibility
**Impact:** User experience, polish, accessibility compliance

### Performance (LOW)
Rules for optimizing build output and runtime performance. Includes content configuration for purging unused styles, JIT mode benefits, and proper usage of arbitrary values.

**Focus:** Bundle size, build time, runtime performance
**Impact:** Load time, user experience, production optimization
