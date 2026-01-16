---
title: Configure HMR for Optimal Development
impact: HIGH
impactDescription: Fast, reliable hot updates
tags: dev, hmr, hot-reload, development, vite
---

## Configure HMR for Optimal Development

**Impact: HIGH (Fast, reliable hot updates)**

Configure Vite's Hot Module Replacement (HMR) for optimal development experience with fast, reliable updates.

## Bad Example

```tsx
// vite.config.ts - No HMR configuration
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  // HMR works with defaults but may have issues in certain environments
});
```

```tsx
// Component that breaks HMR
// UserContext.tsx
let userCache = {}; // Module-level state breaks HMR

export function UserProvider({ children }) {
  const [user, setUser] = useState(() => {
    // Reading from module-level cache during init
    return userCache.current || null;
  });

  useEffect(() => {
    userCache.current = user;
  }, [user]);

  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
}
// Result: HMR causes state loss and unexpected behavior
```

## Good Example

```tsx
// vite.config.ts - Properly configured HMR
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [
    react({
      // Enable Fast Refresh for better HMR
      fastRefresh: true,
    }),
  ],
  server: {
    hmr: {
      // Use overlay for clear error display
      overlay: true,
      // Protocol configuration for specific environments
      protocol: 'ws',
      // Custom port if needed (e.g., behind proxy)
      // port: 24678,
    },
    // Watch configuration
    watch: {
      // Use polling in Docker or network drives
      usePolling: process.env.USE_POLLING === 'true',
      // Ignore node_modules for better performance
      ignored: ['**/node_modules/**', '**/dist/**'],
    },
  },
});
```

```tsx
// vite.config.ts - Docker/WSL optimized HMR
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    host: '0.0.0.0', // Listen on all interfaces in Docker
    hmr: {
      // Connect to host machine from Docker
      host: 'localhost',
      clientPort: 5173, // Exposed port
    },
    watch: {
      // Polling required for Docker volumes
      usePolling: true,
      interval: 1000,
    },
  },
});
```

```tsx
// HMR-compatible state management
// stores/userStore.ts
import { create } from 'zustand';

interface UserState {
  user: User | null;
  setUser: (user: User | null) => void;
}

export const useUserStore = create<UserState>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
}));

// HMR will preserve store state automatically
```

```tsx
// HMR-compatible context with proper boundaries
// contexts/ThemeContext.tsx
import { createContext, useContext, useState, useCallback } from 'react';

interface ThemeContextValue {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextValue | null>(null);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = useCallback(() => {
    setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));
  }, []);

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// Explicitly handle HMR for this module if needed
if (import.meta.hot) {
  import.meta.hot.accept();
}
```

```tsx
// Custom HMR handling for special cases
// utils/apiClient.ts
import axios from 'axios';

export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000,
});

// Handle HMR - recreate interceptors
if (import.meta.hot) {
  import.meta.hot.dispose(() => {
    // Clean up interceptors on module dispose
    apiClient.interceptors.request.clear();
    apiClient.interceptors.response.clear();
  });
}

// Add interceptors
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

```tsx
// vite.config.ts - Full development optimization
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig(({ mode }) => ({
  plugins: [
    react({
      fastRefresh: true,
      // Include emotion or styled-components babel plugins if used
      babel: {
        plugins: mode === 'development' ? ['@emotion/babel-plugin'] : [],
      },
    }),
  ],
  server: {
    hmr: {
      overlay: true,
    },
    watch: {
      // Increase limit for large projects
      ignored: ['**/node_modules/**', '**/dist/**', '**/.git/**'],
    },
  },
  // Optimize dependency pre-bundling for faster HMR
  optimizeDeps: {
    include: [
      'react',
      'react-dom',
      'react-router-dom',
      'zustand',
      '@tanstack/react-query',
    ],
    exclude: ['@vite/client'],
  },
}));
```

## Why

Proper HMR configuration is crucial for developer productivity:

1. **Instant Feedback**: Changes reflect in the browser in milliseconds, not seconds

2. **State Preservation**: React Fast Refresh maintains component state during updates, preserving your development context

3. **Error Visibility**: Clear error overlays help quickly identify and fix issues

4. **Environment Compatibility**: Proper configuration handles Docker, WSL, and network drive scenarios

5. **Memory Management**: Correct HMR cleanup prevents memory leaks during long development sessions

HMR Troubleshooting:

| Issue | Cause | Solution |
|-------|-------|----------|
| Full page reload | Export not a component | Check default exports |
| State lost | Module-level state | Use state management library |
| Changes not detected | File system events | Enable polling |
| Connection errors | Port/protocol mismatch | Configure hmr.clientPort |
| Slow updates | Large dep chain | Optimize with optimizeDeps |

Best Practices:
- Use React Fast Refresh (included in @vitejs/plugin-react)
- Keep components as default exports for best HMR support
- Avoid module-level mutable state
- Use `import.meta.hot` for custom HMR handling when needed
- Enable polling only when necessary (Docker, network drives)
- Pre-bundle frequently used dependencies with `optimizeDeps`
