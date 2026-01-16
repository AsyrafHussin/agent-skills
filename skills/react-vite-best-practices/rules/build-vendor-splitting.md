---
title: Strategic Vendor Code Splitting
impact: CRITICAL
impactDescription: Optimal cache utilization, faster builds
tags: build, vendor, splitting, caching, vite
---

## Strategic Vendor Code Splitting

**Impact: CRITICAL (Optimal cache utilization, faster builds)**

Separate vendor dependencies from application code to optimize caching and reduce rebuild times.

## Bad Example

```tsx
// vite.config.ts - All code in single bundle
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        // No vendor splitting - everything in one file
        manualChunks: undefined,
      },
    },
  },
});
// Result: main-abc123.js (500KB) - changes on every deployment
```

```tsx
// Or all vendors in one giant chunk
manualChunks: {
  vendor: [
    'react', 'react-dom', 'react-router-dom',
    'lodash', 'date-fns', 'axios',
    '@tanstack/react-query', 'zustand',
    'recharts', 'd3', 'monaco-editor',
    // ... 50+ more packages
  ],
}
// Result: vendor-xyz789.js (2MB) - changes when ANY dependency updates
```

## Good Example

```tsx
// vite.config.ts - Strategic vendor splitting
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // Framework core - very stable, rarely changes
          'vendor-react': [
            'react',
            'react-dom',
            'react/jsx-runtime',
          ],

          // Routing - stable, changes occasionally
          'vendor-router': [
            'react-router-dom',
            '@remix-run/router',
          ],

          // State management - moderate change frequency
          'vendor-state': [
            '@tanstack/react-query',
            'zustand',
          ],

          // UI primitives - changes with design system updates
          'vendor-ui': [
            '@radix-ui/react-dialog',
            '@radix-ui/react-dropdown-menu',
            '@radix-ui/react-tooltip',
            '@radix-ui/react-popover',
            'class-variance-authority',
            'clsx',
          ],

          // Utilities - very stable
          'vendor-utils': [
            'lodash-es',
            'date-fns',
            'zod',
          ],
        },
      },
    },
  },
});
```

```tsx
// vite.config.ts - Dynamic vendor splitting with size awareness
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

// Large dependencies that should be isolated
const LARGE_DEPS = new Set([
  'recharts',
  'd3',
  'monaco-editor',
  'pdfjs-dist',
  'xlsx',
  'three',
]);

// Stable core dependencies
const CORE_DEPS = new Set([
  'react',
  'react-dom',
  'react/jsx-runtime',
]);

// Routing dependencies
const ROUTER_DEPS = new Set([
  'react-router-dom',
  'react-router',
  '@remix-run/router',
]);

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: (id) => {
          if (!id.includes('node_modules')) {
            return; // Let app code be handled by code splitting
          }

          // Extract package name
          const match = id.match(/node_modules\/([^/]+)/);
          if (!match) return;

          const packageName = match[1];

          // Core React - most stable
          if (CORE_DEPS.has(packageName)) {
            return 'vendor-react';
          }

          // Router
          if (ROUTER_DEPS.has(packageName)) {
            return 'vendor-router';
          }

          // Large dependencies get their own chunks
          if (LARGE_DEPS.has(packageName)) {
            return `vendor-${packageName}`;
          }

          // Scoped packages grouped by scope
          if (packageName.startsWith('@')) {
            const scope = packageName.split('/')[0];
            return `vendor-${scope}`;
          }

          // Everything else
          return 'vendor-misc';
        },
      },
    },
  },
});
```

```tsx
// vite.config.ts - Production-optimized with dep analysis
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: (id, { getModuleInfo }) => {
          if (!id.includes('node_modules')) return;

          const module = getModuleInfo(id);
          if (!module) return;

          // Check how many other modules import this one
          const importers = module.importers.length;

          // Heavily imported modules go to common chunk
          if (importers > 10) {
            return 'vendor-common';
          }

          // Extract and return appropriate chunk
          const match = id.match(/node_modules\/(@?[^/]+)/);
          if (match) {
            const pkg = match[1].replace('@', '').replace('/', '-');

            // Group by update frequency
            if (['react', 'react-dom'].includes(pkg)) {
              return 'vendor-react';
            }

            return `vendor-${pkg}`;
          }
        },
      },
    },
    chunkSizeWarningLimit: 500,
  },
});
```

## Why

Strategic vendor splitting provides multiple benefits:

1. **Optimal Cache Utilization**: Stable dependencies like React stay cached for months while only application code needs re-downloading on deployments

2. **Faster CI/CD**: Unchanged vendor chunks don't need regeneration, speeding up builds

3. **Parallel Downloads**: Multiple smaller vendor chunks download faster than one large chunk due to HTTP/2 multiplexing

4. **Granular Invalidation**: When you update `lodash`, only that chunk invalidates, not your entire vendor bundle

5. **Better Loading Strategy**: Critical vendor chunks can be preloaded while optional ones (charts, editors) load on demand

Vendor Splitting Strategy:

| Chunk | Contents | Size Target | Cache Duration |
|-------|----------|-------------|----------------|
| vendor-react | React core | ~40KB | 6+ months |
| vendor-router | React Router | ~15KB | 3+ months |
| vendor-state | State management | ~20KB | 1-3 months |
| vendor-ui | UI components | ~50KB | Monthly |
| vendor-utils | Utilities | ~30KB | 6+ months |
| vendor-[large] | Heavy libs | Varies | As needed |

Best Practices:
- Group by update frequency, not functionality
- Isolate large dependencies (>100KB) into their own chunks
- Keep vendor-react separate - it rarely changes
- Use chunk size warnings to identify opportunities for splitting
- Monitor actual chunk sizes with bundle analyzer
