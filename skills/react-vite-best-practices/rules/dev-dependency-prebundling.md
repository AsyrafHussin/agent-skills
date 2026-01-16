---
title: Configure Dependency Pre-bundling
impact: HIGH
impactDescription: 2-5× faster cold start
tags: dev, dependencies, prebundling, optimization, vite
---

## Configure Dependency Pre-bundling

**Impact: HIGH (2-5× faster cold start)**

Vite pre-bundles dependencies to convert CommonJS/UMD to ESM and reduce the number of module requests. Proper configuration speeds up cold starts and prevents runtime issues.

## Incorrect

```typescript
// vite.config.ts
export default defineConfig({
  // No optimizeDeps configuration
  // Vite auto-detects but may miss some deps
})
```

**Problems:**
- Some dependencies may not be pre-bundled
- Cold start can be slow with many dependencies
- Runtime errors from unbundled CommonJS modules

## Correct

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],

  optimizeDeps: {
    // Explicitly include dependencies that should be pre-bundled
    include: [
      'react',
      'react-dom',
      'react-router-dom',
      '@tanstack/react-query',
      'zustand',
      'axios',
      'date-fns',
      // Include nested dependencies if needed
      'react-dom/client',
    ],

    // Exclude dependencies that don't need pre-bundling
    // (already ESM or causing issues)
    exclude: [
      // '@some/esm-only-package',
    ],
  },
})
```

## Force Re-bundling

```typescript
// vite.config.ts
export default defineConfig({
  optimizeDeps: {
    // Force re-bundling (useful when deps update)
    force: true, // Remove after resolving issues
  },
})
```

Or via CLI:
```bash
vite --force
# or
vite optimize --force
```

## Handle CommonJS Dependencies

```typescript
// vite.config.ts
export default defineConfig({
  optimizeDeps: {
    include: [
      // Some packages have deeply nested CommonJS
      'lodash-es',
      // Force include linked packages
      'linked-package > some-dep',
    ],

    // ESBuild options for dependency optimization
    esbuildOptions: {
      // Handle packages that use Node.js globals
      define: {
        global: 'globalThis',
      },
    },
  },
})
```

## Warmup Frequently Used Files

```typescript
// vite.config.ts (Vite 5+)
export default defineConfig({
  server: {
    warmup: {
      // Pre-transform these files on server start
      clientFiles: [
        './src/main.tsx',
        './src/App.tsx',
        './src/components/index.ts',
      ],
    },
  },
})
```

## Debug Pre-bundling

```bash
# See what's being pre-bundled
DEBUG=vite:deps vite

# Check the pre-bundle output
ls node_modules/.vite/deps/
```

## Impact

- 2-5x faster cold start
- Eliminates "optimizing dependencies" during development
- Prevents CommonJS/ESM compatibility issues
