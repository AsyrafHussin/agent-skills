---
title: Preserve Component State with Fast Refresh
impact: MEDIUM
impactDescription: Faster iteration without state loss
tags: hmr, fast-refresh, react, development, state
---

## Preserve Component State with Fast Refresh

**Impact: MEDIUM (Faster iteration without state loss)**

React Fast Refresh preserves component state during hot updates, enabling faster development iteration. Incorrect patterns can break Fast Refresh, causing full reloads and lost state.

## Incorrect

```typescript
// ❌ Mixing exports breaks Fast Refresh
// components/Button.tsx
export function Button() {
  return <button>Click me</button>
}

export const BUTTON_SIZES = { sm: 'small', md: 'medium', lg: 'large' }
export const formatButtonText = (text: string) => text.toUpperCase()
```

```typescript
// ❌ Anonymous default export
export default function() {
  return <div>Anonymous</div>
}

// ❌ Non-component default export
export default {
  title: 'My Component',
  component: MyComponent,
}
```

**Problem:** Fast Refresh only works when a file exports React components exclusively.

## Correct

```typescript
// ✅ Only export components from component files
// components/Button.tsx
export function Button() {
  return <button>Click me</button>
}

// Or default export
export default function Button() {
  return <button>Click me</button>
}
```

```typescript
// ✅ Constants in separate file
// constants/button.ts
export const BUTTON_SIZES = { sm: 'small', md: 'medium', lg: 'large' }
export const formatButtonText = (text: string) => text.toUpperCase()
```

```typescript
// ✅ Named function for default export
export default function MyComponent() {
  return <div>Named</div>
}
```

## Check Fast Refresh Status

```typescript
// In browser console during development
// If you see this warning, Fast Refresh is degraded:
// "[Fast Refresh] performing full reload"

// Check which file caused the issue in terminal output
```

## Common Fast Refresh Breakers

```typescript
// ❌ Class components (prefer function components)
class MyComponent extends React.Component {
  render() {
    return <div />
  }
}

// ❌ Higher-order components in same file
const withAuth = (Component) => {
  return function AuthWrapper(props) {
    return <Component {...props} />
  }
}

export default withAuth(MyComponent) // Breaks Fast Refresh

// ✅ Move HOC to separate file
// hocs/withAuth.tsx
export function withAuth<P>(Component: React.ComponentType<P>) {
  return function AuthWrapper(props: P) {
    return <Component {...props} />
  }
}

// components/MyComponent.tsx
import { withAuth } from '@/hocs/withAuth'

function MyComponent() {
  return <div />
}

export default withAuth(MyComponent)
```

## Vite Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [
    react({
      // Fast Refresh is enabled by default
      fastRefresh: true, // Explicit (optional)
    }),
  ],
  server: {
    hmr: {
      overlay: true, // Show errors in browser
    },
  },
})
```

## Preserve State Across Refreshes

```typescript
// Use key to preserve identity
function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
    </div>
  )
}

// Fast Refresh will preserve count value during edits
```

## Force Full Reload When Needed

```typescript
// Add this comment to force full reload
// @refresh reset

function ComponentThatNeedsFullReload() {
  // Some initialization that needs fresh state
  return <div />
}
```

## Impact

- Instant feedback during development
- State preserved between edits
- Faster iteration cycles
