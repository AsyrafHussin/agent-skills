---
title: Split Large Library Dependencies
impact: CRITICAL
impactDescription: Better caching for third-party code
tags: split, libraries, chunks, vendor, optimization
---

## Split Large Library Dependencies

**Impact: CRITICAL (Better caching for third-party code)**

Configure Vite to split large third-party libraries into separate chunks for optimal caching and loading strategies.

## Bad Example

```tsx
// vite.config.ts - No library chunk configuration
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        // All libraries bundled together
        manualChunks: undefined,
      },
    },
  },
});
// Result: vendor.js contains react, lodash, moment, chart.js, monaco-editor (3MB+)
```

```tsx
// Or naive splitting that creates too many chunks
manualChunks: (id) => {
  if (id.includes('node_modules')) {
    // Every package gets its own chunk - creates hundreds of files
    return id.split('node_modules/')[1].split('/')[0];
  }
}
// Result: 100+ small HTTP requests, negating HTTP/2 benefits
```

## Good Example

```tsx
// vite.config.ts - Strategic library chunk splitting
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

// Define library groups by size and usage patterns
const FRAMEWORK_LIBS = ['react', 'react-dom', 'scheduler'];
const ROUTER_LIBS = ['react-router', 'react-router-dom', '@remix-run/router'];
const STATE_LIBS = ['zustand', '@tanstack/react-query', 'immer'];
const UI_LIBS = ['@radix-ui', '@headlessui', 'framer-motion'];
const FORM_LIBS = ['react-hook-form', '@hookform/resolvers', 'zod'];
const DATE_LIBS = ['date-fns', 'dayjs'];

// Large libraries that should be isolated
const LARGE_LIBS = ['monaco-editor', 'recharts', 'd3', 'three', 'pdfjs-dist'];

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: (id) => {
          if (!id.includes('node_modules')) return;

          // Framework core
          if (FRAMEWORK_LIBS.some(lib => id.includes(`/node_modules/${lib}/`))) {
            return 'lib-react';
          }

          // Router
          if (ROUTER_LIBS.some(lib => id.includes(`/node_modules/${lib}/`) ||
                                       id.includes(`/node_modules/${lib.replace('/', '-')}/`))) {
            return 'lib-router';
          }

          // State management
          if (STATE_LIBS.some(lib => id.includes(`/node_modules/${lib}/`))) {
            return 'lib-state';
          }

          // UI components
          if (UI_LIBS.some(lib => id.includes(`/node_modules/${lib}/`))) {
            return 'lib-ui';
          }

          // Forms
          if (FORM_LIBS.some(lib => id.includes(`/node_modules/${lib}/`))) {
            return 'lib-forms';
          }

          // Date utilities
          if (DATE_LIBS.some(lib => id.includes(`/node_modules/${lib}/`))) {
            return 'lib-dates';
          }

          // Large libraries get their own chunks
          for (const lib of LARGE_LIBS) {
            if (id.includes(`/node_modules/${lib}/`)) {
              return `lib-${lib}`;
            }
          }

          // Remaining node_modules
          return 'lib-vendor';
        },
      },
    },
  },
});
```

```tsx
// vite.config.ts - Size-aware automatic chunking
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: (id, { getModuleInfo }) => {
          if (!id.includes('node_modules')) return;

          const getPackageName = (id: string) => {
            const match = id.match(/node_modules\/(@[^/]+\/[^/]+|[^/]+)/);
            return match ? match[1] : null;
          };

          const packageName = getPackageName(id);
          if (!packageName) return;

          // Always isolate React
          if (['react', 'react-dom'].includes(packageName)) {
            return 'lib-react';
          }

          // Get module info for analysis
          const moduleInfo = getModuleInfo(id);

          // Large modules or heavily imported modules
          if (moduleInfo) {
            const importedCount = moduleInfo.importedIds?.length || 0;

            // Modules with many imports are likely large
            if (importedCount > 50) {
              return `lib-${packageName.replace('@', '').replace('/', '-')}`;
            }
          }

          // Group scoped packages
          if (packageName.startsWith('@')) {
            const scope = packageName.split('/')[0].replace('@', '');

            // Well-known UI scopes
            if (['radix-ui', 'headlessui'].includes(scope)) {
              return 'lib-ui';
            }

            // Tanstack packages
            if (scope === 'tanstack') {
              return 'lib-tanstack';
            }

            return `lib-${scope}`;
          }

          return 'lib-vendor';
        },
      },
    },
    chunkSizeWarningLimit: 300, // Warn for chunks > 300KB
  },
});
```

```tsx
// Package.json scripts for chunk analysis
{
  "scripts": {
    "build": "vite build",
    "build:analyze": "vite build && npx vite-bundle-visualizer",
    "analyze": "npx source-map-explorer dist/assets/*.js"
  }
}

// After running build:analyze, review chunk sizes:
// lib-react.js     ~40KB gzipped (stable)
// lib-router.js    ~12KB gzipped (stable)
// lib-state.js     ~15KB gzipped (moderate changes)
// lib-ui.js        ~45KB gzipped (changes with design)
// lib-forms.js     ~25KB gzipped (stable)
// lib-vendor.js    ~30KB gzipped (catch-all)
// lib-recharts.js  ~80KB gzipped (lazy loaded)
// lib-monaco.js    ~500KB gzipped (lazy loaded)
```

## Why

Strategic library chunking is essential for production performance:

1. **Optimal Cache Granularity**: When you update lodash, users don't need to re-download React. Each library group can be cached independently

2. **Predictable Bundle Sizes**: Grouping by purpose creates consistent chunk sizes, making performance budgets easier to maintain

3. **Loading Prioritization**: Critical libraries (React) can load first while heavy optional libraries (charts) load on demand

4. **HTTP/2 Efficiency**: Modern HTTP/2 handles multiple small requests efficiently, but 5-15 chunks is more optimal than 100+

5. **Build Performance**: Rollup can parallelize chunk generation, speeding up CI/CD pipelines

Library Chunk Strategy:

| Chunk | Contents | Size Target | Priority |
|-------|----------|-------------|----------|
| lib-react | React core | ~40KB | Critical |
| lib-router | Routing | ~15KB | Critical |
| lib-state | State management | ~20KB | High |
| lib-ui | UI primitives | ~50KB | High |
| lib-forms | Form handling | ~25KB | Medium |
| lib-dates | Date utilities | ~10KB | Low |
| lib-vendor | Miscellaneous | <50KB | Low |
| lib-[heavy] | Monaco, D3, etc | Varies | Lazy |

Best Practices:
- Keep critical path libraries in small, separate chunks
- Isolate large libraries (>100KB) that can be lazy loaded
- Group libraries by update frequency for better caching
- Use bundle analyzer to verify chunk sizes
- Set chunk size warnings to catch regressions
