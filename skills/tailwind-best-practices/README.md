# Tailwind CSS Best Practices

Patterns and conventions for effective Tailwind CSS usage.

## Overview

This skill provides guidance for:
- Mobile-first responsive design
- Dark mode implementation
- Component styling patterns
- Tailwind configuration
- Animation and transitions

## Categories

### 1. Responsive Design (Critical)
Mobile-first approach, breakpoints, and responsive layouts.

### 2. Dark Mode (Critical)
Setup, styling, and system preference handling.

### 3. Component Patterns (High)
Conditional classes, variants, and reusable patterns.

### 4. Custom Configuration (High)
Colors, fonts, spacing, and plugins.

### 5. Spacing & Typography (Medium)
Consistent spacing and typography scales.

### 6. Animation (Medium)
Transitions, keyframes, and reduced motion.

### 7. Performance (Low)
Content configuration and optimization.

## Quick Start

```tsx
// cn utility for conditional classes
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}

// Usage
<button
  className={cn(
    'px-4 py-2 rounded-md font-medium',
    variant === 'primary' && 'bg-blue-600 text-white',
    variant === 'secondary' && 'bg-gray-100 text-gray-900',
  )}
>
  Click me
</button>
```

## Usage

This skill triggers automatically when:
- Writing Tailwind classes
- Implementing responsive designs
- Setting up dark mode
- Configuring Tailwind

## References

- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [Tailwind CSS Cheat Sheet](https://nerdcave.com/tailwind-cheat-sheet)
