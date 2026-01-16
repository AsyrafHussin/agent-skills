---
title: Target Modern Browsers for Smaller Bundles
impact: CRITICAL
impactDescription: 10-15% smaller bundles
tags: build, target, modern, es2020, optimization
---

## Target Modern Browsers for Smaller Bundles

**Impact: CRITICAL (10-15% smaller bundles)**

Vite defaults to a broad browser target for compatibility. Modern browsers support ES2020+ features natively. Targeting older browsers includes unnecessary polyfills and transpilation, increasing bundle size.

## Incorrect

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    // Default target includes older browsers
    // Results in larger bundles with polyfills
  },
})
```

Or explicitly targeting old browsers:

```typescript
export default defineConfig({
  build: {
    target: 'es2015', // Too old, includes many polyfills
  },
})
```

## Correct

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  build: {
    // Target modern browsers
    target: 'esnext',

    // Or be specific about browser versions
    // target: ['es2020', 'edge88', 'firefox78', 'chrome87', 'safari14'],
  },
})
```

## With Legacy Browser Support

If you need to support older browsers, use the legacy plugin:

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import legacy from '@vitejs/plugin-legacy'

export default defineConfig({
  plugins: [
    react(),
    legacy({
      targets: ['defaults', 'not IE 11'],
      // Modern chunks for modern browsers
      // Legacy chunks only loaded by old browsers
    }),
  ],
  build: {
    target: 'esnext', // Modern build
  },
})
```

## Target Options

| Target | Use Case |
|--------|----------|
| `esnext` | Latest features, smallest bundle |
| `es2022` | Good balance, wide support |
| `es2020` | Broader support, slightly larger |
| Custom array | Specific browser versions |

## Impact

Targeting `esnext` vs `es2015` can reduce bundle size by 10-30% depending on the codebase.
