---
title: Configure Optimal Minification Settings
impact: CRITICAL
impactDescription: 30-50% smaller bundles
tags: build, minification, optimization, compression, vite
---

## Configure Optimal Minification Settings

**Impact: CRITICAL (30-50% smaller bundles)**

Configure optimal minification settings in Vite to reduce bundle size while maintaining debugging capabilities when needed.

## Bad Example

```tsx
// vite.config.ts - Disabled or suboptimal minification
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    // Minification disabled
    minify: false,
  },
});
```

```tsx
// Or using less efficient minifier without configuration
export default defineConfig({
  plugins: [react()],
  build: {
    minify: 'terser',
    // No terser options configured - uses defaults
  },
});
```

```tsx
// Code patterns that prevent effective minification
// constants.ts
export const CONFIG = {
  API_URL: 'https://api.example.com',
  TIMEOUT: 5000,
  RETRY_COUNT: 3,
};

// component.tsx - Property access prevents minification
function Component() {
  // These property names won't be minified
  return (
    <div>
      <span data-testid="user-name">{user.firstName}</span>
      <span data-testid="user-email">{user.emailAddress}</span>
    </div>
  );
}
```

## Good Example

```tsx
// vite.config.ts - Optimized minification with esbuild (default)
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    // esbuild is the default and fastest option
    minify: 'esbuild',
    // Remove console and debugger in production
    esbuild: {
      drop: ['console', 'debugger'],
      // Keep legal comments
      legalComments: 'none',
    },
  },
});
```

```tsx
// vite.config.ts - Advanced minification with terser for maximum compression
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    minify: 'terser',
    terserOptions: {
      compress: {
        // Remove console.* calls
        drop_console: true,
        // Remove debugger statements
        drop_debugger: true,
        // Inline single-use functions
        inline: 2,
        // Remove unreachable code
        dead_code: true,
        // Optimize boolean expressions
        booleans_as_integers: true,
        // Multiple optimization passes
        passes: 2,
      },
      mangle: {
        // Mangle property names (use with caution)
        properties: {
          // Only mangle properties starting with underscore
          regex: /^_/,
        },
      },
      format: {
        // Remove comments
        comments: false,
        // Produce ASCII output
        ascii_only: true,
      },
    },
  },
});
```

```tsx
// Code patterns that support effective minification
// Use private class fields for better mangling
class UserService {
  #apiClient;
  #cache = new Map();

  constructor(apiClient: ApiClient) {
    this.#apiClient = apiClient;
  }

  async #fetchUser(id: string) {
    if (this.#cache.has(id)) {
      return this.#cache.get(id);
    }
    const user = await this.#apiClient.get(`/users/${id}`);
    this.#cache.set(id, user);
    return user;
  }

  getUser(id: string) {
    return this.#fetchUser(id);
  }
}
```

```tsx
// Environment-aware console removal
// logger.ts
const isDev = import.meta.env.DEV;

export const logger = {
  log: isDev ? console.log.bind(console) : () => {},
  warn: isDev ? console.warn.bind(console) : () => {},
  error: console.error.bind(console), // Keep errors in production
};

// Usage - logs are removed in production
import { logger } from './logger';

function processData(data: Data) {
  logger.log('Processing:', data);
  // ...
  return result;
}
```

## Why

Proper minification is essential for production applications:

1. **Significant Size Reduction**: Minification typically reduces JavaScript bundle size by 50-70%, directly improving load times

2. **Faster Parse Time**: Shorter variable names and removed whitespace mean browsers can parse the code faster

3. **Bandwidth Savings**: Smaller files reduce server bandwidth costs and improve performance on slow connections

4. **Code Obfuscation**: While not a security measure, minification makes reverse engineering slightly harder

5. **Console Cleanup**: Removing console statements prevents information leakage and improves runtime performance

Minification Options Compared:
- **esbuild** (Vite default): Extremely fast, good compression, ideal for development and most production builds
- **terser**: Slower but produces slightly smaller bundles (2-5% smaller), better for maximum optimization

Best Practices:
- Use esbuild for faster builds during development
- Consider terser for production if every KB matters
- Remove console/debugger statements in production
- Use private class fields (`#`) for better property mangling
- Avoid patterns that prevent minification (string property access, `eval`)
