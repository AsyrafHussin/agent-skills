---
title: Configure Build for Effective Tree Shaking
impact: CRITICAL
impactDescription: 15-30% smaller bundles
tags: build, tree-shaking, optimization, dead-code, vite
---

## Configure Build for Effective Tree Shaking

**Impact: CRITICAL (15-30% smaller bundles)**

Configure your Vite build to effectively eliminate dead code through tree shaking, reducing bundle size significantly.

## Bad Example

```tsx
// utils/index.ts - Barrel export that prevents tree shaking
export * from './strings';
export * from './numbers';
export * from './dates';
export * from './arrays';
export * from './objects';

// Using namespace imports
import * as utils from './utils';

function Component() {
  // Only using one function but importing everything
  return <div>{utils.formatDate(new Date())}</div>;
}
```

```tsx
// Importing entire libraries
import _ from 'lodash';
import moment from 'moment';

function processData(items: Item[]) {
  // Using only 2 functions but importing entire library
  return _.uniqBy(items, 'id').map(item => ({
    ...item,
    date: moment(item.date).format('YYYY-MM-DD'),
  }));
}
```

```json
// package.json - Missing sideEffects field
{
  "name": "my-app",
  "version": "1.0.0",
  "main": "dist/index.js",
  "module": "dist/index.esm.js"
}
```

## Good Example

```tsx
// utils/index.ts - Named exports for better tree shaking
export { formatString, capitalize, truncate } from './strings';
export { formatNumber, clamp, round } from './numbers';
export { formatDate, parseDate, isValidDate } from './dates';
export { unique, groupBy, sortBy } from './arrays';
export { pick, omit, merge } from './objects';

// Direct named imports
import { formatDate } from './utils';

function Component() {
  return <div>{formatDate(new Date())}</div>;
}
```

```tsx
// Import only what you need from tree-shakeable libraries
import uniqBy from 'lodash-es/uniqBy';
import { format } from 'date-fns';

function processData(items: Item[]) {
  return uniqBy(items, 'id').map(item => ({
    ...item,
    date: format(new Date(item.date), 'yyyy-MM-dd'),
  }));
}
```

```json
// package.json - Proper sideEffects configuration
{
  "name": "my-app",
  "version": "1.0.0",
  "main": "dist/index.js",
  "module": "dist/index.esm.js",
  "sideEffects": [
    "*.css",
    "*.scss",
    "./src/polyfills.ts"
  ]
}
```

```tsx
// vite.config.ts - Optimize dependencies for tree shaking
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      treeshake: {
        moduleSideEffects: 'no-external',
        propertyReadSideEffects: false,
        tryCatchDeoptimization: false,
      },
    },
  },
  optimizeDeps: {
    include: ['lodash-es'],
  },
});
```

## Why

Tree shaking is a critical optimization technique that:

1. **Dramatically Reduces Bundle Size**: Unused exports are eliminated from the final bundle. A library might be 100KB but you only include the 5KB you actually use

2. **Improves Load Performance**: Smaller bundles mean faster downloads, especially on mobile networks

3. **Enables Modular Architecture**: You can organize code in feature-rich modules without worrying about bloating the bundle

4. **Works with ES Modules**: Tree shaking relies on static analysis of ES module imports/exports, which is why ESM-compatible libraries like `lodash-es` and `date-fns` are preferred

5. **Compounds with Code Splitting**: Combined with code splitting, tree shaking ensures each chunk contains only the code it needs

Key practices for effective tree shaking:
- Use ES modules (`import`/`export`) instead of CommonJS (`require`/`module.exports`)
- Prefer libraries that ship ES module builds
- Avoid namespace imports (`import * as`)
- Configure `sideEffects` in package.json to help bundlers identify pure modules
- Use named exports instead of default exports where possible
