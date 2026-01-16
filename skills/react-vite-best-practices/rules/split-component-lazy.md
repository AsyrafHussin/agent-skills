---
title: Lazy Load Non-Critical Components
impact: CRITICAL
impactDescription: 20-40% smaller initial bundle
tags: split, lazy, components, code-splitting, react
---

## Lazy Load Non-Critical Components

**Impact: CRITICAL (20-40% smaller initial bundle)**

Use React.lazy for component-level code splitting to load non-critical UI components on demand.

## Bad Example

```tsx
// Dashboard.tsx - All components imported eagerly
import { useState } from 'react';
import Header from './components/Header';
import Sidebar from './components/Sidebar';
import MainContent from './components/MainContent';
import SettingsPanel from './components/SettingsPanel';
import NotificationCenter from './components/NotificationCenter';
import UserProfileModal from './components/UserProfileModal';
import HelpDrawer from './components/HelpDrawer';
import FeedbackForm from './components/FeedbackForm';
import AdvancedFilters from './components/AdvancedFilters';
import ExportDialog from './components/ExportDialog';
import ChartWidget from './components/ChartWidget';
import DataTable from './components/DataTable';

function Dashboard() {
  const [showSettings, setShowSettings] = useState(false);
  const [showProfile, setShowProfile] = useState(false);
  const [showHelp, setShowHelp] = useState(false);
  const [showFeedback, setShowFeedback] = useState(false);
  const [showFilters, setShowFilters] = useState(false);
  const [showExport, setShowExport] = useState(false);

  return (
    <div>
      <Header />
      <Sidebar />
      <MainContent />
      {showSettings && <SettingsPanel />}
      {showProfile && <UserProfileModal />}
      {showHelp && <HelpDrawer />}
      {showFeedback && <FeedbackForm />}
      {showFilters && <AdvancedFilters />}
      {showExport && <ExportDialog />}
    </div>
  );
}
// Result: All modals, drawers, and dialogs loaded even if never opened
```

## Good Example

```tsx
// Dashboard.tsx - Component-level lazy loading
import { lazy, Suspense, useState } from 'react';
import Header from './components/Header';
import Sidebar from './components/Sidebar';
import MainContent from './components/MainContent';
import { Skeleton } from './components/ui/Skeleton';

// Lazy load components that aren't immediately visible
const SettingsPanel = lazy(() => import('./components/SettingsPanel'));
const NotificationCenter = lazy(() => import('./components/NotificationCenter'));
const UserProfileModal = lazy(() => import('./components/UserProfileModal'));
const HelpDrawer = lazy(() => import('./components/HelpDrawer'));
const FeedbackForm = lazy(() => import('./components/FeedbackForm'));
const AdvancedFilters = lazy(() => import('./components/AdvancedFilters'));
const ExportDialog = lazy(() => import('./components/ExportDialog'));

// Reusable component for lazy-loaded modals
function LazyModal({
  isOpen,
  children
}: {
  isOpen: boolean;
  children: React.ReactNode
}) {
  if (!isOpen) return null;

  return (
    <Suspense fallback={<Skeleton className="modal-skeleton" />}>
      {children}
    </Suspense>
  );
}

function Dashboard() {
  const [showSettings, setShowSettings] = useState(false);
  const [showProfile, setShowProfile] = useState(false);
  const [showHelp, setShowHelp] = useState(false);
  const [showFeedback, setShowFeedback] = useState(false);
  const [showFilters, setShowFilters] = useState(false);
  const [showExport, setShowExport] = useState(false);

  return (
    <div>
      <Header
        onSettingsClick={() => setShowSettings(true)}
        onProfileClick={() => setShowProfile(true)}
      />
      <Sidebar />
      <MainContent />

      <LazyModal isOpen={showSettings}>
        <SettingsPanel onClose={() => setShowSettings(false)} />
      </LazyModal>

      <LazyModal isOpen={showProfile}>
        <UserProfileModal onClose={() => setShowProfile(false)} />
      </LazyModal>

      <LazyModal isOpen={showHelp}>
        <HelpDrawer onClose={() => setShowHelp(false)} />
      </LazyModal>

      <LazyModal isOpen={showFeedback}>
        <FeedbackForm onClose={() => setShowFeedback(false)} />
      </LazyModal>

      <LazyModal isOpen={showFilters}>
        <AdvancedFilters onClose={() => setShowFilters(false)} />
      </LazyModal>

      <LazyModal isOpen={showExport}>
        <ExportDialog onClose={() => setShowExport(false)} />
      </LazyModal>
    </div>
  );
}
```

```tsx
// Advanced: Lazy component with preloading and error handling
// utils/lazyWithPreload.tsx
import { lazy, ComponentType, LazyExoticComponent } from 'react';

interface PreloadableComponent<T extends ComponentType<any>>
  extends LazyExoticComponent<T> {
  preload: () => Promise<{ default: T }>;
}

export function lazyWithPreload<T extends ComponentType<any>>(
  factory: () => Promise<{ default: T }>
): PreloadableComponent<T> {
  const Component = lazy(factory) as PreloadableComponent<T>;
  Component.preload = factory;
  return Component;
}

// Usage
const SettingsPanel = lazyWithPreload(() => import('./components/SettingsPanel'));
const ExportDialog = lazyWithPreload(() => import('./components/ExportDialog'));

// Preload on hover
function SettingsButton({ onClick }: { onClick: () => void }) {
  return (
    <button
      onClick={onClick}
      onMouseEnter={() => SettingsPanel.preload()}
      onFocus={() => SettingsPanel.preload()}
    >
      Settings
    </button>
  );
}
```

```tsx
// Lazy loading below-the-fold content
// pages/ProductPage.tsx
import { lazy, Suspense } from 'react';
import { useInView } from 'react-intersection-observer';
import ProductHeader from './components/ProductHeader';
import ProductGallery from './components/ProductGallery';
import ProductDetails from './components/ProductDetails';

// Heavy components below the fold
const RelatedProducts = lazy(() => import('./components/RelatedProducts'));
const CustomerReviews = lazy(() => import('./components/CustomerReviews'));
const SimilarItems = lazy(() => import('./components/SimilarItems'));

function ProductPage({ productId }: { productId: string }) {
  const { ref: reviewsRef, inView: reviewsInView } = useInView({
    triggerOnce: true,
    rootMargin: '200px', // Load 200px before entering viewport
  });

  const { ref: relatedRef, inView: relatedInView } = useInView({
    triggerOnce: true,
    rootMargin: '200px',
  });

  return (
    <div>
      {/* Critical above-the-fold content */}
      <ProductHeader productId={productId} />
      <ProductGallery productId={productId} />
      <ProductDetails productId={productId} />

      {/* Lazy loaded below-the-fold content */}
      <section ref={reviewsRef}>
        {reviewsInView && (
          <Suspense fallback={<ReviewsSkeleton />}>
            <CustomerReviews productId={productId} />
          </Suspense>
        )}
      </section>

      <section ref={relatedRef}>
        {relatedInView && (
          <Suspense fallback={<ProductGridSkeleton />}>
            <RelatedProducts productId={productId} />
            <SimilarItems productId={productId} />
          </Suspense>
        )}
      </section>
    </div>
  );
}
```

## Why

Component-level lazy loading provides fine-grained control over when code is loaded:

1. **Reduced Initial Bundle**: Modals, drawers, and dialogs that users may never open don't bloat the initial download

2. **Faster First Paint**: Critical UI renders quickly while non-essential components load in the background

3. **User-Centric Loading**: Code is fetched based on user actions, not developer assumptions about what might be needed

4. **Better Memory Usage**: Components and their dependencies only occupy memory when actually rendered

5. **Improved Mobile Experience**: Especially important on slower devices where parsing large bundles blocks the main thread

When to Lazy Load Components:
| Component Type | Lazy Load? | Reason |
|---------------|------------|--------|
| Modals/Dialogs | Yes | Only shown on interaction |
| Drawers/Panels | Yes | Hidden by default |
| Below-fold content | Yes | Not in initial viewport |
| Tabs (non-default) | Yes | Hidden until selected |
| Admin features | Yes | Limited user base |
| Header/Navigation | No | Always visible |
| Above-fold content | No | Critical for FCP |

Best Practices:
- Lazy load all modal and drawer content
- Use intersection observer for below-the-fold components
- Implement preloading on hover for smoother UX
- Keep Suspense fallbacks lightweight (skeletons, not spinners)
- Group related lazy components to minimize HTTP requests
