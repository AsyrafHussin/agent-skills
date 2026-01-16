---
title: Design Optimal Chunk Splitting Strategy
impact: CRITICAL
impactDescription: Balance caching and loading performance
tags: build, chunks, strategy, optimization, splitting
---

## Design Optimal Chunk Splitting Strategy

**Impact: CRITICAL (Balance caching and loading performance)**

Design an optimal chunk splitting strategy that balances caching efficiency with loading performance.

## Bad Example

```tsx
// vite.config.ts - No strategic chunk planning
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    // Results in either one huge bundle or too many small chunks
    rollupOptions: {
      output: {
        // Random chunking without strategy
        manualChunks: (id) => {
          if (id.includes('node_modules')) {
            return 'vendor'; // All vendors in one huge chunk
          }
        },
      },
    },
  },
});
```

```tsx
// Or the opposite extreme - too granular
manualChunks: (id) => {
  if (id.includes('node_modules')) {
    // Creates hundreds of tiny chunks
    return id.split('node_modules/')[1].split('/')[0];
  }
},
```

## Good Example

```tsx
// vite.config.ts - Strategic chunk configuration
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: (id) => {
          // Core React runtime - rarely changes
          if (id.includes('node_modules/react/') ||
              id.includes('node_modules/react-dom/') ||
              id.includes('node_modules/scheduler/')) {
            return 'react-core';
          }

          // Router - changes occasionally
          if (id.includes('node_modules/react-router') ||
              id.includes('node_modules/@remix-run/router')) {
            return 'router';
          }

          // State management - moderate change frequency
          if (id.includes('node_modules/zustand') ||
              id.includes('node_modules/@tanstack/react-query')) {
            return 'state';
          }

          // UI component library - changes with design updates
          if (id.includes('node_modules/@radix-ui') ||
              id.includes('node_modules/@headlessui')) {
            return 'ui-components';
          }

          // Utility libraries - stable
          if (id.includes('node_modules/lodash') ||
              id.includes('node_modules/date-fns') ||
              id.includes('node_modules/zod')) {
            return 'utils';
          }

          // Charts and visualization - large, used on specific pages
          if (id.includes('node_modules/recharts') ||
              id.includes('node_modules/d3')) {
            return 'charts';
          }

          // Remaining node_modules
          if (id.includes('node_modules/')) {
            return 'vendor';
          }
        },
      },
    },
    // Warn if chunks exceed reasonable size
    chunkSizeWarningLimit: 250,
  },
});
```

```tsx
// Advanced: Function-based chunking with size awareness
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

const LARGE_DEPS = ['recharts', 'd3', 'monaco-editor', 'pdf-lib'];
const CORE_DEPS = ['react', 'react-dom', 'react-router-dom'];

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: (id, { getModuleInfo }) => {
          if (!id.includes('node_modules')) return;

          const moduleInfo = getModuleInfo(id);

          // Check if this is a large dependency that should be isolated
          for (const dep of LARGE_DEPS) {
            if (id.includes(`node_modules/${dep}`)) {
              return `vendor-${dep}`;
            }
          }

          // Core dependencies in one chunk
          for (const dep of CORE_DEPS) {
            if (id.includes(`node_modules/${dep}`)) {
              return 'vendor-core';
            }
          }

          // Group remaining by scope
          const match = id.match(/node_modules\/(@[^/]+\/[^/]+|[^/]+)/);
          if (match) {
            const pkgName = match[1];
            // Group scoped packages together
            if (pkgName.startsWith('@')) {
              const scope = pkgName.split('/')[0];
              return `vendor-${scope}`;
            }
          }

          return 'vendor-misc';
        },
      },
    },
  },
});
```

## Why

A well-designed chunk strategy is crucial for optimal application performance:

1. **Maximizes Cache Efficiency**: By grouping dependencies by change frequency, you ensure that stable code (like React itself) stays cached while frequently updated code can be refreshed without re-downloading everything

2. **Balances HTTP Requests vs Size**: Too few chunks mean large downloads; too many chunks increase HTTP overhead. The sweet spot is typically 5-15 chunks for most applications

3. **Optimizes Critical Path**: Core chunks needed for initial render can be prioritized while feature-specific chunks (charts, editors) load on demand

4. **Improves Update Experience**: When you deploy updates, users only re-download the changed chunks, making subsequent visits fast

5. **Enables Parallel Loading**: Multiple smaller chunks can be downloaded in parallel, better utilizing available bandwidth

Chunk Strategy Guidelines:
- **Core chunks (< 50KB gzipped)**: React, React DOM, router
- **Feature chunks (50-150KB gzipped)**: State management, UI library
- **Large library chunks**: Isolate libraries > 100KB (charts, editors, PDF)
- **Application chunks**: Split by route or feature
- **Shared chunks**: Common components used across multiple routes
