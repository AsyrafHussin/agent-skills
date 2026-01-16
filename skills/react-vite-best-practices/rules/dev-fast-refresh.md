---
title: Structure Components for Fast Refresh
impact: HIGH
impactDescription: Instant updates without losing state
tags: dev, fast-refresh, hmr, react, development
---

## Structure Components for Fast Refresh

**Impact: HIGH (Instant updates without losing state)**

Structure components to take full advantage of React Fast Refresh for instant updates during development.

## Bad Example

```tsx
// App.tsx - Patterns that break Fast Refresh

// Named exports can break Fast Refresh in some cases
export const App = () => {
  return <div>App</div>;
};

// Multiple component exports in one file
export const Header = () => <header>Header</header>;
export const Footer = () => <footer>Footer</footer>;
export const Sidebar = () => <aside>Sidebar</aside>;
```

```tsx
// UserProfile.tsx - Component with side effects at module level
import { fetchUser } from './api';

// Side effect at module level - breaks Fast Refresh
const initialUser = await fetchUser('current');

export default function UserProfile() {
  const [user] = useState(initialUser);
  return <div>{user.name}</div>;
}
```

```tsx
// Counter.tsx - Mixing components with non-component exports
export default function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  );
}

// Non-component export in same file - may break Fast Refresh
export const MAX_COUNT = 100;
export const formatCount = (n: number) => n.toLocaleString();
```

```tsx
// Anonymous component - Fast Refresh can't identify it
export default function() {
  return <div>Anonymous</div>;
}

// Arrow function without name
export default () => {
  return <div>Also anonymous</div>;
};
```

## Good Example

```tsx
// App.tsx - Default export for main component
export default function App() {
  return (
    <div>
      <Header />
      <main>
        <Outlet />
      </main>
      <Footer />
    </div>
  );
}
```

```tsx
// components/Header.tsx - One component per file
export default function Header() {
  const { user } = useAuth();

  return (
    <header className="header">
      <Logo />
      <Navigation />
      <UserMenu user={user} />
    </header>
  );
}
```

```tsx
// constants/counter.ts - Separate file for constants
export const MAX_COUNT = 100;
export const MIN_COUNT = 0;
export const STEP = 1;

// utils/format.ts - Separate file for utilities
export function formatCount(n: number): string {
  return n.toLocaleString();
}

// components/Counter.tsx - Pure component file
import { useState } from 'react';
import { MAX_COUNT, MIN_COUNT, STEP } from '../constants/counter';
import { formatCount } from '../utils/format';

export default function Counter() {
  const [count, setCount] = useState(0);

  const increment = () => {
    setCount((c) => Math.min(c + STEP, MAX_COUNT));
  };

  const decrement = () => {
    setCount((c) => Math.max(c - STEP, MIN_COUNT));
  };

  return (
    <div className="counter">
      <button onClick={decrement}>-</button>
      <span>{formatCount(count)}</span>
      <button onClick={increment}>+</button>
    </div>
  );
}
```

```tsx
// UserProfile.tsx - Proper data fetching pattern
import { useQuery } from '@tanstack/react-query';
import { fetchUser } from '../api/users';
import { Skeleton } from './ui/Skeleton';

export default function UserProfile() {
  const { data: user, isLoading, error } = useQuery({
    queryKey: ['user', 'current'],
    queryFn: () => fetchUser('current'),
  });

  if (isLoading) return <Skeleton />;
  if (error) return <ErrorMessage error={error} />;

  return (
    <div className="user-profile">
      <Avatar src={user.avatar} alt={user.name} />
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

```tsx
// hooks/useCounter.ts - Custom hooks in separate files
import { useState, useCallback } from 'react';

interface UseCounterOptions {
  initialValue?: number;
  min?: number;
  max?: number;
  step?: number;
}

export function useCounter(options: UseCounterOptions = {}) {
  const { initialValue = 0, min = -Infinity, max = Infinity, step = 1 } = options;

  const [count, setCount] = useState(initialValue);

  const increment = useCallback(() => {
    setCount((c) => Math.min(c + step, max));
  }, [step, max]);

  const decrement = useCallback(() => {
    setCount((c) => Math.max(c - step, min));
  }, [step, min]);

  const reset = useCallback(() => {
    setCount(initialValue);
  }, [initialValue]);

  return { count, increment, decrement, reset, setCount };
}

// components/Counter.tsx - Component using the hook
import { useCounter } from '../hooks/useCounter';

export default function Counter() {
  const { count, increment, decrement, reset } = useCounter({
    min: 0,
    max: 100,
  });

  return (
    <div>
      <button onClick={decrement}>-</button>
      <span>{count}</span>
      <button onClick={increment}>+</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

```tsx
// Higher-order components - Preserve display names
import { ComponentType } from 'react';

export function withAuth<P extends object>(
  WrappedComponent: ComponentType<P>
) {
  function WithAuth(props: P) {
    const { user, isLoading } = useAuth();

    if (isLoading) return <LoadingSpinner />;
    if (!user) return <Navigate to="/login" />;

    return <WrappedComponent {...props} />;
  }

  // Important: Set display name for Fast Refresh and DevTools
  WithAuth.displayName = `WithAuth(${
    WrappedComponent.displayName || WrappedComponent.name || 'Component'
  })`;

  return WithAuth;
}
```

## Why

React Fast Refresh provides instant feedback during development, but requires specific patterns:

1. **State Preservation**: Fast Refresh keeps component state intact during edits, so you don't lose form inputs or scroll position

2. **Quick Iteration**: Changes reflect in ~50ms, enabling rapid UI development and experimentation

3. **Error Recovery**: When errors occur, fixing them restores the previous state without full reload

4. **Accurate Updates**: Only changed components re-render, maintaining the accuracy of your development view

5. **Better DX**: Developers can focus on code changes without managing browser state

Fast Refresh Requirements:

| Pattern | Fast Refresh | Notes |
|---------|--------------|-------|
| Default export function | Works | Recommended |
| Named export function | Usually works | Name must be PascalCase |
| Anonymous function | Fails | Always name components |
| Multiple components/file | May break | One component per file |
| Non-component exports | May break | Separate into utility files |
| Class components | Limited | Function components preferred |

Best Practices:
- One React component per file
- Use default exports for components
- Always name your components (no anonymous functions)
- Keep constants and utilities in separate files
- Use hooks for data fetching instead of module-level side effects
- Set displayName on HOCs and forwardRef components
