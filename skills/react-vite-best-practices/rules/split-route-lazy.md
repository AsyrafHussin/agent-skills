---
title: Use React.lazy() for Route-Based Splitting
impact: CRITICAL
impactDescription: 50-80% smaller initial bundle
tags: split, lazy, routes, code-splitting, react
---

## Use React.lazy() for Route-Based Splitting

**Impact: CRITICAL (50-80% smaller initial bundle)**

Loading all route components upfront delays initial page load. Users download code for pages they may never visit. Route-based code splitting ensures users only download code for the current route.

## Incorrect

```typescript
// App.tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom'

// All imports are eager - loaded immediately
import Home from './pages/Home'
import Dashboard from './pages/Dashboard'
import Settings from './pages/Settings'
import Profile from './pages/Profile'
import Admin from './pages/Admin'

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
        <Route path="/profile" element={<Profile />} />
        <Route path="/admin" element={<Admin />} />
      </Routes>
    </BrowserRouter>
  )
}
```

**Problem:** All 5 page components are bundled together and loaded on initial page load, even if user only visits the home page.

## Correct

```typescript
// App.tsx
import { lazy, Suspense } from 'react'
import { BrowserRouter, Routes, Route } from 'react-router-dom'

// Lazy load route components
const Home = lazy(() => import('./pages/Home'))
const Dashboard = lazy(() => import('./pages/Dashboard'))
const Settings = lazy(() => import('./pages/Settings'))
const Profile = lazy(() => import('./pages/Profile'))
const Admin = lazy(() => import('./pages/Admin'))

// Loading component
function PageLoader() {
  return (
    <div className="flex items-center justify-center min-h-screen">
      <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600" />
    </div>
  )
}

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/settings" element={<Settings />} />
          <Route path="/profile" element={<Profile />} />
          <Route path="/admin" element={<Admin />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  )
}
```

## With Named Chunks

```typescript
// Better debugging with named chunks
const Dashboard = lazy(() =>
  import(/* webpackChunkName: "dashboard" */ './pages/Dashboard')
)

// Vite native way (recommended)
const Settings = lazy(() => import('./pages/Settings'))
// Vite automatically names chunks based on file path
```

## With Preloading

```typescript
// Preload on hover for instant navigation
const Dashboard = lazy(() => import('./pages/Dashboard'))

function NavLink() {
  const preloadDashboard = () => {
    import('./pages/Dashboard')
  }

  return (
    <Link
      to="/dashboard"
      onMouseEnter={preloadDashboard}
      onFocus={preloadDashboard}
    >
      Dashboard
    </Link>
  )
}
```

## Impact

- Initial bundle can be reduced by 50-80%
- Time to Interactive significantly improved
- Each route loads only when needed
