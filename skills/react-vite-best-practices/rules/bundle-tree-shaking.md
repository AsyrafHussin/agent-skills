---
title: Ensure Proper Tree Shaking with ESM Imports
impact: MEDIUM
impactDescription: 20-40% smaller bundle with proper imports
tags: bundle, tree-shaking, esm, optimization, imports
---

## Ensure Proper Tree Shaking with ESM Imports

**Impact: MEDIUM (20-40% smaller bundle with proper imports)**

Tree shaking removes unused code from bundles. Improper imports can prevent tree shaking, including entire libraries when only small parts are used.

## Incorrect

```typescript
// ❌ Imports entire library
import _ from 'lodash'
const result = _.get(obj, 'path')

// ❌ Namespace import prevents tree shaking
import * as utils from './utils'
utils.formatDate(date)

// ❌ Importing from barrel file
import { Button } from '@/components'
// If components/index.ts exports 50 components,
// all may be included
```

## Correct

```typescript
// ✅ Import only what you need
import get from 'lodash/get'
const result = get(obj, 'path')

// ✅ Or use lodash-es for better tree shaking
import { get } from 'lodash-es'

// ✅ Named imports allow tree shaking
import { formatDate } from './utils'
formatDate(date)

// ✅ Direct imports from source
import { Button } from '@/components/Button'
```

## Avoid Barrel Files for Large Libraries

```typescript
// ❌ components/index.ts (barrel file)
export * from './Button'
export * from './Input'
export * from './Modal'
export * from './Table'
export * from './Chart'
// ... 50 more components

// ❌ Consumer imports one, gets all
import { Button } from '@/components'
```

```typescript
// ✅ Direct imports
import { Button } from '@/components/Button'
import { Input } from '@/components/Input'

// ✅ Or use a smaller barrel for related components
// components/forms/index.ts
export { Input } from './Input'
export { Select } from './Select'
export { Checkbox } from './Checkbox'
```

## Check Tree Shaking with Visualizer

```bash
npm install rollup-plugin-visualizer -D
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import { visualizer } from 'rollup-plugin-visualizer'

export default defineConfig({
  plugins: [
    visualizer({
      filename: 'dist/stats.html',
      open: true,
      gzipSize: true,
    }),
  ],
})
```

## Side Effects Configuration

```json
// package.json
{
  "sideEffects": false
}

// Or specify files with side effects
{
  "sideEffects": [
    "*.css",
    "*.scss",
    "./src/polyfills.ts"
  ]
}
```

## Common Tree Shaking Issues

```typescript
// ❌ Default exports can be harder to tree shake
export default {
  formatDate,
  formatCurrency,
  formatNumber,
}

// ✅ Named exports tree shake better
export { formatDate }
export { formatCurrency }
export { formatNumber }
```

```typescript
// ❌ Re-exporting without type annotation
export { User } from './types'

// ✅ Type-only exports are removed
export type { User } from './types'
```

## Vite Configuration

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      // Treat these as external (not bundled)
      external: ['react', 'react-dom'],

      // Tree shake properly
      treeshake: {
        moduleSideEffects: false,
        propertyReadSideEffects: false,
      },
    },
  },
})
```

## Impact

- 10-50% smaller bundles depending on imports
- Faster load times
- Better caching (smaller chunks change less)
