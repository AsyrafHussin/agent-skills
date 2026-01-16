---
title: Implement Route-Based Code Splitting
impact: CRITICAL
impactDescription: 50-70% smaller initial bundle
tags: split, lazy, routes, code-splitting, react-router
---

## Implement Route-Based Code Splitting

**Impact: CRITICAL (50-70% smaller initial bundle)**

Implement route-based code splitting using React.lazy to load route components on demand, reducing initial bundle size.

## Bad Example

```tsx
// App.tsx - All routes imported eagerly
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import Dashboard from './pages/Dashboard';
import Analytics from './pages/Analytics';
import Reports from './pages/Reports';
import Settings from './pages/Settings';
import UserManagement from './pages/UserManagement';
import Billing from './pages/Billing';
import AuditLog from './pages/AuditLog';
import Integrations from './pages/Integrations';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Dashboard />} />
        <Route path="/analytics" element={<Analytics />} />
        <Route path="/reports" element={<Reports />} />
        <Route path="/settings" element={<Settings />} />
        <Route path="/users" element={<UserManagement />} />
        <Route path="/billing" element={<Billing />} />
        <Route path="/audit" element={<AuditLog />} />
        <Route path="/integrations" element={<Integrations />} />
      </Routes>
    </BrowserRouter>
  );
}
// Result: All page code downloaded on initial load (~800KB)
```

## Good Example

```tsx
// App.tsx - Lazy loaded routes with Suspense
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { PageLoader } from './components/PageLoader';

// Lazy load all route components
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Analytics = lazy(() => import('./pages/Analytics'));
const Reports = lazy(() => import('./pages/Reports'));
const Settings = lazy(() => import('./pages/Settings'));
const UserManagement = lazy(() => import('./pages/UserManagement'));
const Billing = lazy(() => import('./pages/Billing'));
const AuditLog = lazy(() => import('./pages/AuditLog'));
const Integrations = lazy(() => import('./pages/Integrations'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/" element={<Dashboard />} />
          <Route path="/analytics" element={<Analytics />} />
          <Route path="/reports" element={<Reports />} />
          <Route path="/settings" element={<Settings />} />
          <Route path="/users" element={<UserManagement />} />
          <Route path="/billing" element={<Billing />} />
          <Route path="/audit" element={<AuditLog />} />
          <Route path="/integrations" element={<Integrations />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
// Result: Only Dashboard loaded initially (~150KB), others on navigation
```

```tsx
// routes/index.tsx - Organized lazy routes with named chunks
import { lazy, Suspense } from 'react';
import { RouteObject } from 'react-router-dom';
import { PageLoader } from '../components/PageLoader';

// Named chunks for better debugging and analysis
const Dashboard = lazy(() =>
  import(/* webpackChunkName: "dashboard" */ '../pages/Dashboard')
);
const Analytics = lazy(() =>
  import(/* webpackChunkName: "analytics" */ '../pages/Analytics')
);
const Reports = lazy(() =>
  import(/* webpackChunkName: "reports" */ '../pages/Reports')
);

// Group related routes
const SettingsLayout = lazy(() =>
  import(/* webpackChunkName: "settings" */ '../pages/Settings/Layout')
);
const GeneralSettings = lazy(() =>
  import(/* webpackChunkName: "settings" */ '../pages/Settings/General')
);
const SecuritySettings = lazy(() =>
  import(/* webpackChunkName: "settings" */ '../pages/Settings/Security')
);

// Helper to wrap components with Suspense
function lazyRoute(Component: React.LazyExoticComponent<any>) {
  return (
    <Suspense fallback={<PageLoader />}>
      <Component />
    </Suspense>
  );
}

export const routes: RouteObject[] = [
  { path: '/', element: lazyRoute(Dashboard) },
  { path: '/analytics', element: lazyRoute(Analytics) },
  { path: '/reports', element: lazyRoute(Reports) },
  {
    path: '/settings',
    element: lazyRoute(SettingsLayout),
    children: [
      { path: 'general', element: lazyRoute(GeneralSettings) },
      { path: 'security', element: lazyRoute(SecuritySettings) },
    ],
  },
];
```

```tsx
// Advanced: Lazy routes with preloading on hover
// routes/LazyRoute.tsx
import { lazy, Suspense, ComponentType, LazyExoticComponent } from 'react';
import { PageLoader } from '../components/PageLoader';

type LazyFactory = () => Promise<{ default: ComponentType<any> }>;

interface PreloadableLazyComponent extends LazyExoticComponent<ComponentType<any>> {
  preload: LazyFactory;
}

// Create a lazy component with preload capability
export function createLazyRoute(factory: LazyFactory): PreloadableLazyComponent {
  const Component = lazy(factory) as PreloadableLazyComponent;
  Component.preload = factory;
  return Component;
}

// Usage
export const Dashboard = createLazyRoute(() => import('../pages/Dashboard'));
export const Analytics = createLazyRoute(() => import('../pages/Analytics'));

// Link component with preload on hover
// components/PreloadLink.tsx
import { Link, LinkProps } from 'react-router-dom';
import { useCallback } from 'react';

interface PreloadLinkProps extends LinkProps {
  preload?: () => Promise<any>;
}

export function PreloadLink({ preload, onMouseEnter, ...props }: PreloadLinkProps) {
  const handleMouseEnter = useCallback((e: React.MouseEvent<HTMLAnchorElement>) => {
    preload?.();
    onMouseEnter?.(e);
  }, [preload, onMouseEnter]);

  return <Link {...props} onMouseEnter={handleMouseEnter} />;
}

// In navigation
import { PreloadLink } from './PreloadLink';
import { Dashboard, Analytics } from '../routes';

function Navigation() {
  return (
    <nav>
      <PreloadLink to="/" preload={Dashboard.preload}>
        Dashboard
      </PreloadLink>
      <PreloadLink to="/analytics" preload={Analytics.preload}>
        Analytics
      </PreloadLink>
    </nav>
  );
}
```

## Why

Route-based lazy loading is the most impactful code splitting strategy:

1. **Dramatically Smaller Initial Bundle**: Users download only the code for the current page. A 10-page app might load 80% less JavaScript initially

2. **Faster Time to Interactive**: Less JavaScript to parse and execute means users can interact with the page sooner

3. **Natural Code Boundaries**: Routes provide clear splitting points that align with user navigation patterns

4. **Improved Core Web Vitals**: Smaller initial bundles directly improve LCP, FID, and TTI metrics

5. **Progressive Enhancement**: The app shell loads immediately while route-specific code loads in the background

Best Practices:
- Lazy load all routes except the landing page (if it's critical)
- Use named chunks for easier debugging and bundle analysis
- Implement preloading on link hover for perceived instant navigation
- Nest Suspense boundaries strategically - one for the entire routes or per-route for finer control
- Consider loading indicators that match your app's design language

Route Splitting Impact:
| App Size | Eager Loading | Lazy Loading | Improvement |
|----------|---------------|--------------|-------------|
| Small (5 pages) | 200KB | 80KB | 60% |
| Medium (15 pages) | 500KB | 120KB | 76% |
| Large (30+ pages) | 1.2MB | 180KB | 85% |
