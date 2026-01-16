# State Management (React Query + Zustand)

Patterns for server state (React Query) and client state (Zustand).

## Overview

This skill provides guidance for:
- Data fetching with React Query
- Caching and invalidation strategies
- Mutations and optimistic updates
- Client state with Zustand
- Combining both libraries

## Categories

### 1. React Query Basics (Critical)
Setup, useQuery, query keys, and error handling.

### 2. Zustand Store Patterns (Critical)
Store creation, TypeScript, selectors, and persistence.

### 3. Caching & Invalidation (High)
Stale time, gc time, invalidation, and prefetching.

### 4. Mutations & Updates (High)
useMutation, callbacks, and cache updates.

### 5. Optimistic Updates (Medium)
Optimistic UI updates with rollback.

### 6. DevTools & Debugging (Medium)
Development tools for both libraries.

## Server State vs Client State

| Server State (React Query) | Client State (Zustand) |
|---------------------------|----------------------|
| Data from APIs | UI state |
| Cached remotely | Local only |
| May be stale | Always current |
| Needs refetching | No fetching |
| Examples: users, posts | Examples: modals, themes |

## Quick Start

```tsx
// React Query: Server state
const { data, isLoading } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
})

// Zustand: Client state
const useUIStore = create((set) => ({
  isModalOpen: false,
  openModal: () => set({ isModalOpen: true }),
  closeModal: () => set({ isModalOpen: false }),
}))
```

## Usage

This skill triggers automatically when:
- Fetching data from APIs
- Managing application state
- Implementing caching
- Handling mutations

## References

- [TanStack Query Documentation](https://tanstack.com/query)
- [Zustand Documentation](https://zustand-demo.pmnd.rs/)
