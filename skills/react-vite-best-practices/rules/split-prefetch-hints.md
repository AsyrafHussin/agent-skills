---
title: Prefetch Code Chunks on User Intent
impact: MEDIUM
impactDescription: Instant navigation perceived speed
tags: split, prefetch, preload, performance, code-splitting
---

## Prefetch Code Chunks on User Intent

**Impact: MEDIUM (Instant navigation perceived speed)**

Use prefetch and preload hints to load code chunks before they're needed, improving perceived navigation speed.

## Bad Example

```tsx
// No prefetching - chunks load only when navigation occurs
import { lazy, Suspense } from 'react';
import { Routes, Route, Link } from 'react-router-dom';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Analytics = lazy(() => import('./pages/Analytics'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <>
      <nav>
        <Link to="/">Dashboard</Link>
        <Link to="/analytics">Analytics</Link>
        <Link to="/settings">Settings</Link>
      </nav>

      <Suspense fallback={<Loading />}>
        <Routes>
          <Route path="/" element={<Dashboard />} />
          <Route path="/analytics" element={<Analytics />} />
          <Route path="/settings" element={<Settings />} />
        </Routes>
      </Suspense>
    </>
  );
}
// Result: User clicks link -> waits for chunk to download -> sees loading -> page renders
```

## Good Example

```tsx
// Prefetch on hover/focus for instant-feeling navigation
import { lazy, Suspense, useCallback } from 'react';
import { Routes, Route, Link, LinkProps } from 'react-router-dom';

// Create lazy components with preload capability
function lazyWithPreload<T extends React.ComponentType<any>>(
  factory: () => Promise<{ default: T }>
) {
  const Component = lazy(factory);
  (Component as any).preload = factory;
  return Component as typeof Component & { preload: typeof factory };
}

const Dashboard = lazyWithPreload(() => import('./pages/Dashboard'));
const Analytics = lazyWithPreload(() => import('./pages/Analytics'));
const Settings = lazyWithPreload(() => import('./pages/Settings'));

// Link component that prefetches on hover
interface PrefetchLinkProps extends LinkProps {
  preload?: () => Promise<any>;
}

function PrefetchLink({ preload, onMouseEnter, onFocus, ...props }: PrefetchLinkProps) {
  const handlePreload = useCallback(() => {
    preload?.();
  }, [preload]);

  return (
    <Link
      {...props}
      onMouseEnter={(e) => {
        handlePreload();
        onMouseEnter?.(e);
      }}
      onFocus={(e) => {
        handlePreload();
        onFocus?.(e);
      }}
    />
  );
}

function App() {
  return (
    <>
      <nav>
        <PrefetchLink to="/" preload={Dashboard.preload}>
          Dashboard
        </PrefetchLink>
        <PrefetchLink to="/analytics" preload={Analytics.preload}>
          Analytics
        </PrefetchLink>
        <PrefetchLink to="/settings" preload={Settings.preload}>
          Settings
        </PrefetchLink>
      </nav>

      <Suspense fallback={<Loading />}>
        <Routes>
          <Route path="/" element={<Dashboard />} />
          <Route path="/analytics" element={<Analytics />} />
          <Route path="/settings" element={<Settings />} />
        </Routes>
      </Suspense>
    </>
  );
}
// Result: User hovers link -> chunk downloads -> user clicks -> instant navigation
```

```tsx
// Advanced: Prefetch based on viewport visibility
// components/PrefetchOnVisible.tsx
import { useEffect, useRef } from 'react';

interface PrefetchOnVisibleProps {
  children: React.ReactNode;
  preload: () => Promise<any>;
  rootMargin?: string;
}

export function PrefetchOnVisible({
  children,
  preload,
  rootMargin = '200px',
}: PrefetchOnVisibleProps) {
  const ref = useRef<HTMLDivElement>(null);
  const prefetched = useRef(false);

  useEffect(() => {
    if (!ref.current || prefetched.current) return;

    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting && !prefetched.current) {
          prefetched.current = true;
          preload();
          observer.disconnect();
        }
      },
      { rootMargin }
    );

    observer.observe(ref.current);

    return () => observer.disconnect();
  }, [preload, rootMargin]);

  return <div ref={ref}>{children}</div>;
}

// Usage: Prefetch analytics when footer becomes visible
function Footer() {
  return (
    <PrefetchOnVisible preload={Analytics.preload}>
      <footer>
        <Link to="/analytics">View Analytics</Link>
      </footer>
    </PrefetchOnVisible>
  );
}
```

```tsx
// Prefetch after idle time
// hooks/usePrefetchAfterIdle.ts
import { useEffect, useRef } from 'react';

export function usePrefetchAfterIdle(
  preloadFns: Array<() => Promise<any>>,
  delay: number = 2000
) {
  const prefetched = useRef(false);

  useEffect(() => {
    if (prefetched.current) return;

    const prefetch = () => {
      if (prefetched.current) return;
      prefetched.current = true;

      // Prefetch with low priority
      preloadFns.forEach((fn) => {
        if ('requestIdleCallback' in window) {
          requestIdleCallback(() => fn(), { timeout: 5000 });
        } else {
          setTimeout(fn, 100);
        }
      });
    };

    // Wait for initial load, then prefetch during idle
    const timeoutId = setTimeout(prefetch, delay);

    return () => clearTimeout(timeoutId);
  }, [preloadFns, delay]);
}

// Usage in App component
function App() {
  // Prefetch common routes 2 seconds after initial load
  usePrefetchAfterIdle([
    Analytics.preload,
    Settings.preload,
  ], 2000);

  return (/* ... */);
}
```

```tsx
// Vite-specific: Use modulepreload for critical chunks
// index.html
<!DOCTYPE html>
<html>
<head>
  <!-- Preload critical vendor chunks -->
  <link rel="modulepreload" href="/assets/lib-react.js" />
  <link rel="modulepreload" href="/assets/lib-router.js" />

  <!-- Prefetch likely-to-be-needed chunks (lower priority) -->
  <link rel="prefetch" href="/assets/analytics.js" />
  <link rel="prefetch" href="/assets/settings.js" />
</head>
<body>
  <div id="root"></div>
  <script type="module" src="/src/main.tsx"></script>
</body>
</html>

// vite.config.ts - Generate modulepreload automatically
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    modulePreload: {
      // Customize which chunks to preload
      polyfill: true,
      resolveDependencies: (filename, deps, { hostId, hostType }) => {
        // Only preload critical dependencies
        return deps.filter(dep =>
          dep.includes('lib-react') ||
          dep.includes('lib-router')
        );
      },
    },
  },
});
```

## Why

Prefetch hints dramatically improve perceived performance:

1. **Instant-Feeling Navigation**: Code loads while users decide, making clicks feel instantaneous

2. **Better User Experience**: Eliminates loading spinners for common navigation paths

3. **Efficient Bandwidth Usage**: Prefetching happens during idle time, not competing with critical resources

4. **Maintains Code Splitting Benefits**: You still get smaller initial bundles, just with smarter preloading

5. **Predictable Performance**: Users on slow connections benefit the most from preloading

Prefetch Strategies:

| Strategy | Trigger | Best For |
|----------|---------|----------|
| Hover/Focus | User intent signal | Navigation links |
| Viewport Entry | Scroll position | Below-fold sections |
| Idle Time | After initial load | Common routes |
| Route Matching | Current route | Related pages |
| modulepreload | Page load | Critical vendors |

Priority Guidelines:
- `preload`: Critical resources needed immediately
- `prefetch`: Resources likely needed for next navigation
- `modulepreload`: ES modules that should be parsed early

Best Practices:
- Prefetch on hover for navigation items
- Use `requestIdleCallback` for non-critical prefetching
- Don't over-prefetch - prioritize likely navigation paths
- Consider user's data saver preferences
- Monitor network waterfall to verify prefetch timing
