---
title: Strategic Suspense Boundaries for Lazy Loading
impact: CRITICAL
impactDescription: Progressive loading, better UX
tags: split, suspense, lazy, react, boundaries
---

## Strategic Suspense Boundaries for Lazy Loading

**Impact: CRITICAL (Progressive loading, better UX)**

Without proper Suspense boundaries, a single lazy component can block the entire UI. Strategic placement of Suspense boundaries allows parts of the UI to load independently.

## Incorrect

```typescript
// Single Suspense at root - entire app shows loading state
function App() {
  return (
    <Suspense fallback={<FullPageLoader />}>
      <Header />
      <Sidebar />
      <MainContent />
      <Footer />
    </Suspense>
  )
}
```

**Problem:** If any lazy component is loading, the entire app shows the loading state.

## Correct

```typescript
// Strategic Suspense boundaries
function App() {
  return (
    <div className="app-layout">
      {/* Header loads immediately - not lazy */}
      <Header />

      <div className="main-layout">
        {/* Sidebar has its own boundary */}
        <Suspense fallback={<SidebarSkeleton />}>
          <Sidebar />
        </Suspense>

        {/* Main content independent */}
        <Suspense fallback={<ContentSkeleton />}>
          <MainContent />
        </Suspense>
      </div>

      {/* Footer loads immediately */}
      <Footer />
    </div>
  )
}
```

## Nested Suspense for Complex UIs

```typescript
function Dashboard() {
  return (
    <div className="dashboard">
      <h1>Dashboard</h1>

      <div className="dashboard-grid">
        {/* Each widget loads independently */}
        <Suspense fallback={<WidgetSkeleton />}>
          <StatsWidget />
        </Suspense>

        <Suspense fallback={<WidgetSkeleton />}>
          <ChartWidget />
        </Suspense>

        <Suspense fallback={<WidgetSkeleton />}>
          <RecentActivityWidget />
        </Suspense>
      </div>
    </div>
  )
}
```

## Error Boundaries with Suspense

```typescript
import { ErrorBoundary } from 'react-error-boundary'

function App() {
  return (
    <ErrorBoundary fallback={<ErrorFallback />}>
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </ErrorBoundary>
  )
}

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div className="error-container">
      <h2>Something went wrong</h2>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  )
}
```

## Skeleton Components

```typescript
// Good skeleton matches actual content layout
function ContentSkeleton() {
  return (
    <div className="animate-pulse">
      <div className="h-8 bg-gray-200 rounded w-1/4 mb-4" />
      <div className="h-4 bg-gray-200 rounded w-full mb-2" />
      <div className="h-4 bg-gray-200 rounded w-full mb-2" />
      <div className="h-4 bg-gray-200 rounded w-3/4" />
    </div>
  )
}
```

## Benefits

- Parts of UI render independently
- Better perceived performance
- Graceful degradation on slow networks
