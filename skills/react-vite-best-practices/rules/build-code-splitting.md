---
title: Configure Automatic Code Splitting
impact: CRITICAL
impactDescription: Better caching, faster initial loads
tags: build, code-splitting, chunks, optimization, vite
---

## Configure Automatic Code Splitting

**Impact: CRITICAL (Better caching, faster initial loads)**

Configure Vite to automatically split your application code into smaller chunks for better caching and faster initial loads.

## Bad Example

```tsx
// vite.config.ts - No code splitting configuration
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    // All code bundled into a single file
    rollupOptions: {
      output: {
        manualChunks: undefined,
      },
    },
  },
});
```

```tsx
// App.tsx - All imports at top level
import Dashboard from './pages/Dashboard';
import Settings from './pages/Settings';
import Profile from './pages/Profile';
import Analytics from './pages/Analytics';
import Reports from './pages/Reports';

function App() {
  return (
    <Routes>
      <Route path="/dashboard" element={<Dashboard />} />
      <Route path="/settings" element={<Settings />} />
      <Route path="/profile" element={<Profile />} />
      <Route path="/analytics" element={<Analytics />} />
      <Route path="/reports" element={<Reports />} />
    </Routes>
  );
}
```

## Good Example

```tsx
// vite.config.ts - Proper code splitting configuration
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // Vendor chunk for React ecosystem
          'react-vendor': ['react', 'react-dom', 'react-router-dom'],
          // UI library chunk
          'ui-vendor': ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu'],
          // Utility libraries chunk
          'utils-vendor': ['lodash-es', 'date-fns', 'zod'],
        },
      },
    },
    // Generate chunk size warnings
    chunkSizeWarningLimit: 500,
  },
});
```

```tsx
// App.tsx - Lazy loaded routes
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';
import { LoadingSpinner } from './components/LoadingSpinner';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));
const Profile = lazy(() => import('./pages/Profile'));
const Analytics = lazy(() => import('./pages/Analytics'));
const Reports = lazy(() => import('./pages/Reports'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
        <Route path="/profile" element={<Profile />} />
        <Route path="/analytics" element={<Analytics />} />
        <Route path="/reports" element={<Reports />} />
      </Routes>
    </Suspense>
  );
}
```

## Why

Code splitting is essential for modern web applications because:

1. **Faster Initial Load**: Users download only the code needed for the current page, reducing Time to Interactive (TTI)

2. **Better Caching**: Smaller, separate chunks can be cached independently. When you update one part of your app, users only need to re-download that specific chunk

3. **Parallel Downloads**: Browsers can download multiple smaller files simultaneously, utilizing available bandwidth more efficiently

4. **Reduced Memory Usage**: Loading code on-demand means less JavaScript needs to be parsed and executed upfront

5. **Improved Core Web Vitals**: Smaller initial bundles directly improve Largest Contentful Paint (LCP) and First Input Delay (FID) metrics

Vite's built-in Rollup configuration makes code splitting straightforward with `manualChunks` for vendor libraries and dynamic imports for route-based splitting.
