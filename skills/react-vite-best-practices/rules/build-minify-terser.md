---
title: Use Terser for Production Minification
impact: CRITICAL
impactDescription: 5-10% smaller bundles
tags: build, minification, terser, optimization, vite
---

## Use Terser for Production Minification

**Impact: CRITICAL (5-10% smaller bundles)**

Vite uses esbuild for minification by default (fast but less optimal). Terser produces smaller bundles through advanced optimizations like dead code elimination and console removal.

## Incorrect

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    // Default esbuild minification
    // No console removal
    // Less aggressive optimization
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
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,     // Remove console.log
        drop_debugger: true,    // Remove debugger
        pure_funcs: [           // Remove specific functions
          'console.info',
          'console.debug',
          'console.warn',
        ],
      },
      mangle: {
        safari10: true,         // Safari 10 compatibility
      },
      format: {
        comments: false,        // Remove all comments
      },
    },
  },
})
```

## Keep Error Logging

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: false,    // Keep console
        pure_funcs: [
          'console.log',        // Remove only log
          'console.debug',
          'console.info',
          // Keep console.error and console.warn for debugging
        ],
      },
    },
  },
})
```

## Development vs Production

```typescript
// vite.config.ts
export default defineConfig(({ mode }) => ({
  build: {
    minify: mode === 'production' ? 'terser' : 'esbuild',
    terserOptions: mode === 'production' ? {
      compress: {
        drop_console: true,
        drop_debugger: true,
      },
    } : undefined,
  },
}))
```

## Comparison

| Feature | esbuild | Terser |
|---------|---------|--------|
| Speed | Very Fast | Slower |
| Bundle Size | Good | Better (5-10% smaller) |
| Console Removal | Manual | Built-in |
| Dead Code | Basic | Advanced |

## When to Use Each

- **esbuild:** Development, fast builds, CI where speed matters
- **Terser:** Production builds, when bundle size is critical

## Impact

- 5-10% smaller production bundles
- No debug code in production
- Better dead code elimination
