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
