---
title: Configure Manual Chunks for Vendor Separation
impact: CRITICAL
impactDescription: Optimal caching and parallel loading
tags: build, chunks, vendor, optimization, rollup
---

## Configure Manual Chunks for Vendor Separation

**Impact: CRITICAL (Optimal caching and parallel loading)**

Without manual chunks, Vite bundles all vendor dependencies into a single chunk or mixes them with application code. This leads to:
- Large initial bundle downloads
- Poor cache efficiency (vendor code changes with app code)
- Slower subsequent page loads

## Incorrect

```typescript
// vite.config.ts
export default defineConfig({
  plugins: [react()],
  build: {
    // No manual chunks configured
    // All code bundled together
  },
})
```

**Problem:** React, React DOM, and other vendors are bundled with your application code. When you update your app, users must re-download everything.

## Correct

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // Core React - rarely changes
          'vendor-react': ['react', 'react-dom'],

          // Router - changes occasionally
          'vendor-router': ['react-router-dom'],

          // UI library - if using one
          // 'vendor-ui': ['@headlessui/react', '@heroicons/react'],

          // State management
          // 'vendor-state': ['zustand', '@tanstack/react-query'],
        },
      },
    },
  },
})
```

## Advanced: Dynamic Manual Chunks

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks(id) {
          // Node modules go to vendor chunk
          if (id.includes('node_modules')) {
            // Split large libraries into separate chunks
            if (id.includes('react-dom')) {
              return 'vendor-react-dom'
            }
            if (id.includes('react')) {
              return 'vendor-react'
            }
            if (id.includes('@tanstack')) {
              return 'vendor-tanstack'
            }
            // Other node_modules
            return 'vendor'
          }
        },
      },
    },
  },
})
```

## Benefits

- **Better caching:** Vendor chunks cached separately from app code
- **Parallel loading:** Browser can download multiple chunks simultaneously
- **Smaller updates:** App changes don't invalidate vendor cache
