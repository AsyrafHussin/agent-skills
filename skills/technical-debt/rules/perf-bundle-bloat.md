---
title: Frontend Bundle Bloat
impact: HIGH
impactDescription: "Bundle size directly drives bounce rate, INP, and conversion"
tags: performance, frontend, bundle-size
---

## Frontend Bundle Bloat

**Impact: HIGH (Bundle size directly drives bounce rate, INP, and conversion)**

Every kilobyte the browser must parse and execute slows down first paint, interaction-readiness, and on mobile networks, page abandonment. Bundle bloat creeps in invisibly — a moment-import here, a `lodash` import there, an unused route bundled with the entry point — and the team only notices when Lighthouse drops a grade.

## How to Detect

```bash
# Vite
npx vite-bundle-visualizer

# Webpack
npx webpack-bundle-analyzer dist/stats.json

# Any bundler — source map visualization
npx source-map-explorer 'dist/**/*.js'

# CI bundle-size budget (size-limit)
npm install --save-dev size-limit @size-limit/preset-app
# package.json:
#   "size-limit": [{ "path": "dist/index.js", "limit": "200 KB" }]
npx size-limit
```

Budget targets (gzipped, mobile-first):
- **Initial bundle:** < 200 KB
- **Route bundles:** < 100 KB
- **Single dep:** anything > 50 KB deserves justification

## Incorrect

```typescript
// ❌ Default import → ships the entire library
import _ from 'lodash';                     // ~70 KB min / ~24 KB gzip
import moment from 'moment';                // ~290 KB min / ~70 KB gzip with locales

const debounced = _.debounce(handler, 200);
const formatted = moment().format('YYYY-MM-DD');

// ❌ No code splitting — admin pages bundled with the public site
import AdminDashboard from './admin/Dashboard';
import AdminUsers from './admin/Users';
// ... all eagerly imported in the entry file
```

**Problems:**
- Importing all of lodash to use `debounce` is like buying a truck to carry one tomato
- Moment with all locales ships ~70 KB gzip of timezone data nobody uses
- Admin code bundled with the public site triples the entry-bundle for 99% of visitors who never visit `/admin`

## Correct

```typescript
// ✅ Named imports / smaller libraries
import { debounce } from 'lodash-es';        // tree-shaken via ESM
import { format } from 'date-fns';           // ~10 KB gzip for the single `format` import

const debounced = debounce(handler, 200);
const formatted = format(new Date(), 'yyyy-MM-dd');

// ✅ Route-level code splitting
const AdminDashboard = lazy(() => import('./admin/Dashboard'));
const AdminUsers     = lazy(() => import('./admin/Users'));

// In your router:
<Route path="/admin" element={<Suspense fallback={<Spinner />}><AdminDashboard /></Suspense>} />
```

Add a CI guard:

```yaml
- name: Bundle-size budget
  run: npx size-limit
# Fails the build if any tracked bundle exceeds its budget
```

**Benefits:**
- Initial bundle shrinks dramatically; LCP and INP both improve
- Admin code only loads for admin users
- A regression (someone adds `import * as everything`) fails CI

## Remediation Strategy

- **Effort:** S–M per dep (swap imports, lazy-load routes)
- **When to pay down:**
  - **First:** run the bundle visualizer and target the top 5 contributors
  - **Then:** install a bundle-size budget in CI to prevent regression
  - **Ongoing:** every PR that adds a dep > 20KB should be reviewed for alternatives
- **Common wins:**
  - `moment` → `date-fns` or `dayjs` (–80–90% size)
  - `lodash` → `lodash-es` + named imports, or native equivalents
  - Route-level code splitting (10× reduction on admin-heavy apps)
  - Drop polyfills for unsupported browsers
  - Replace heavy SVG icon sets with on-demand icon components

Reference: [web.dev — Apply Instant Loading](https://web.dev/articles/apply-instant-loading-with-prpl) · [BundlePhobia](https://bundlephobia.com/) · [size-limit](https://github.com/ai/size-limit)
