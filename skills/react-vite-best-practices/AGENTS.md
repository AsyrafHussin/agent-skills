# React + Vite Best Practices

**Version 1.0.0**  
agent-skills  
January 2026

> **Note:**  
> This document is for AI agents and LLMs when maintaining, generating, or  
> refactoring React + Vite codebases. Humans may also find it useful, but  
> guidance here is optimized for automation and consistency by AI-assisted workflows.

---

## Abstract

Comprehensive performance optimization guide for React applications built with Vite, designed for AI agents and LLMs. Contains 26 rules across 8 categories, prioritized by impact from critical (build optimization, code splitting) to incremental (advanced patterns). Each rule includes detailed explanations, real-world examples comparing incorrect vs. correct implementations, and specific impact metrics to guide automated refactoring and code generation.

---

## Table of Contents

1. [Build Optimization](#1-build-optimization) — **CRITICAL**
2. [Code Splitting](#2-code-splitting) — **CRITICAL**
3. [Development Performance](#3-development-performance) — **HIGH**
4. [Asset Handling](#4-asset-handling) — **HIGH**
5. [Environment Configuration](#5-environment-configuration) — **MEDIUM**
6. [HMR Optimization](#6-hmr-optimization) — **MEDIUM**
7. [Bundle Analysis](#7-bundle-analysis) — **LOW-MEDIUM**
8. [Advanced Patterns](#8-advanced-patterns) — **LOW**

---

## 1. Build Optimization

**Impact: CRITICAL**

### 1.1 Configure Asset Hashing for Cache Busting

## Configure Asset Hashing for Cache Busting

**Impact: CRITICAL (Ensures latest version delivery)**

Configure content-based asset hashing to enable aggressive caching while ensuring users always receive the latest version after deployments.

## Bad Example

```tsx
// vite.config.ts - No hash configuration or predictable naming
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        // No hash - files get cached indefinitely
        entryFileNames: 'assets/[name].js',
        chunkFileNames: 'assets/[name].js',
        assetFileNames: 'assets/[name].[ext]',
      },
    },
  },
});
```

```html
<!-- index.html - Cache problems -->
<script src="/assets/main.js"></script>
<link rel="stylesheet" href="/assets/style.css">
<!-- Users might see stale content after deployments -->
```

```tsx
// Version-based hashing (bad - all files invalidated on any change)
output: {
  entryFileNames: `assets/[name].${packageJson.version}.js`,
  chunkFileNames: `assets/[name].${packageJson.version}.js`,
  assetFileNames: `assets/[name].${packageJson.version}.[ext]`,
}
```

## Good Example

```tsx
// vite.config.ts - Content-based hashing (Vite default behavior, enhanced)
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        // Content hash ensures unique URLs when content changes
        entryFileNames: 'assets/js/[name]-[hash].js',
        chunkFileNames: 'assets/js/[name]-[hash].js',
        assetFileNames: (assetInfo) => {
          // Organize assets by type
          const info = assetInfo.name?.split('.') || [];
          const ext = info[info.length - 1];

          if (/png|jpe?g|gif|svg|webp|avif|ico/i.test(ext)) {
            return 'assets/images/[name]-[hash][extname]';
          }
          if (/woff2?|eot|ttf|otf/i.test(ext)) {
            return 'assets/fonts/[name]-[hash][extname]';
          }
          if (/css/i.test(ext)) {
            return 'assets/css/[name]-[hash][extname]';
          }
          return 'assets/[name]-[hash][extname]';
        },
      },
    },
  },
});
```

```tsx
// vite.config.ts - Short hashes for cleaner URLs (optional)
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        // 8-character hash is sufficient for most apps
        hashCharacters: 'base36',
        entryFileNames: 'js/[name].[hash:8].js',
        chunkFileNames: 'js/[name].[hash:8].js',
        assetFileNames: '[ext]/[name].[hash:8].[ext]',
      },
    },
  },
});
```

```tsx
// Server caching configuration
// server.ts with Express
import express from 'express';
import path from 'path';

const app = express();

// Immutable caching for hashed assets (1 year)
app.use('/assets', express.static(path.join(__dirname, 'dist/assets'), {
  maxAge: '1y',
  immutable: true,
}));

// Short cache for index.html (always check for updates)
app.use(express.static(path.join(__dirname, 'dist'), {
  maxAge: '5m',
  setHeaders: (res, path) => {
    if (path.endsWith('.html')) {
      // HTML files should be revalidated
      res.setHeader('Cache-Control', 'no-cache, must-revalidate');
    }
  },
}));
```

```nginx
# nginx.conf - Optimal caching strategy
server {
    listen 80;
    root /var/www/app/dist;

    # HTML files - always validate
    location ~* \.html$ {
        add_header Cache-Control "no-cache, must-revalidate";
        add_header Vary "Accept-Encoding";
        try_files $uri /index.html;
    }

    # Hashed assets - cache forever
    location /assets/ {
        add_header Cache-Control "public, max-age=31536000, immutable";
        add_header Vary "Accept-Encoding";
        try_files $uri =404;
    }

    # Service worker - short cache
    location = /sw.js {
        add_header Cache-Control "no-cache, must-revalidate";
        try_files $uri =404;
    }
}
```

## Why

Content-based asset hashing is fundamental to modern web application deployment:

1. **Cache Invalidation Solved**: When file content changes, the hash changes, creating a new URL that bypasses cached versions automatically

2. **Aggressive Caching**: Hashed files can be cached indefinitely with `immutable` directive since the URL changes with the content

3. **Instant Updates**: Users receive new code immediately after deployment without clearing their cache

4. **Bandwidth Efficiency**: Unchanged files remain cached while only updated files are downloaded

5. **CDN Compatibility**: Content hashes work perfectly with CDNs and edge caching strategies

Hashing Strategies:
| Type | Example | Pros | Cons |
|------|---------|------|------|
| Content Hash | `main-a1b2c3d4.js` | Only changes when content changes | Perfect for caching |
| Version Hash | `main-1.0.0.js` | Predictable | Invalidates all files |
| No Hash | `main.js` | Simple | Cache invalidation issues |

Cache-Control Headers:
- **Hashed assets**: `Cache-Control: public, max-age=31536000, immutable`
- **HTML files**: `Cache-Control: no-cache, must-revalidate`
- **Service workers**: `Cache-Control: no-cache, must-revalidate`

Best Practices:
- Use content hashes (not version numbers) for cache busting
- Set immutable caching for hashed assets
- Never cache HTML files - they contain references to hashed assets
- Organize assets by type for easier server configuration
- Consider shorter hashes (8 chars) for cleaner URLs without sacrificing uniqueness


### 1.2 Design Optimal Chunk Splitting Strategy

## Design Optimal Chunk Splitting Strategy

**Impact: CRITICAL (Balance caching and loading performance)**

Design an optimal chunk splitting strategy that balances caching efficiency with loading performance.

## Bad Example

```tsx
// vite.config.ts - No strategic chunk planning
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    // Results in either one huge bundle or too many small chunks
    rollupOptions: {
      output: {
        // Random chunking without strategy
        manualChunks: (id) => {
          if (id.includes('node_modules')) {
            return 'vendor'; // All vendors in one huge chunk
          }
        },
      },
    },
  },
});
```

```tsx
// Or the opposite extreme - too granular
manualChunks: (id) => {
  if (id.includes('node_modules')) {
    // Creates hundreds of tiny chunks
    return id.split('node_modules/')[1].split('/')[0];
  }
},
```

## Good Example

```tsx
// vite.config.ts - Strategic chunk configuration
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: (id) => {
          // Core React runtime - rarely changes
          if (id.includes('node_modules/react/') ||
              id.includes('node_modules/react-dom/') ||
              id.includes('node_modules/scheduler/')) {
            return 'react-core';
          }

          // Router - changes occasionally
          if (id.includes('node_modules/react-router') ||
              id.includes('node_modules/@remix-run/router')) {
            return 'router';
          }

          // State management - moderate change frequency
          if (id.includes('node_modules/zustand') ||
              id.includes('node_modules/@tanstack/react-query')) {
            return 'state';
          }

          // UI component library - changes with design updates
          if (id.includes('node_modules/@radix-ui') ||
              id.includes('node_modules/@headlessui')) {
            return 'ui-components';
          }

          // Utility libraries - stable
          if (id.includes('node_modules/lodash') ||
              id.includes('node_modules/date-fns') ||
              id.includes('node_modules/zod')) {
            return 'utils';
          }

          // Charts and visualization - large, used on specific pages
          if (id.includes('node_modules/recharts') ||
              id.includes('node_modules/d3')) {
            return 'charts';
          }

          // Remaining node_modules
          if (id.includes('node_modules/')) {
            return 'vendor';
          }
        },
      },
    },
    // Warn if chunks exceed reasonable size
    chunkSizeWarningLimit: 250,
  },
});
```

```tsx
// Advanced: Function-based chunking with size awareness
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

const LARGE_DEPS = ['recharts', 'd3', 'monaco-editor', 'pdf-lib'];
const CORE_DEPS = ['react', 'react-dom', 'react-router-dom'];

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: (id, { getModuleInfo }) => {
          if (!id.includes('node_modules')) return;

          const moduleInfo = getModuleInfo(id);

          // Check if this is a large dependency that should be isolated
          for (const dep of LARGE_DEPS) {
            if (id.includes(`node_modules/${dep}`)) {
              return `vendor-${dep}`;
            }
          }

          // Core dependencies in one chunk
          for (const dep of CORE_DEPS) {
            if (id.includes(`node_modules/${dep}`)) {
              return 'vendor-core';
            }
          }

          // Group remaining by scope
          const match = id.match(/node_modules\/(@[^/]+\/[^/]+|[^/]+)/);
          if (match) {
            const pkgName = match[1];
            // Group scoped packages together
            if (pkgName.startsWith('@')) {
              const scope = pkgName.split('/')[0];
              return `vendor-${scope}`;
            }
          }

          return 'vendor-misc';
        },
      },
    },
  },
});
```

## Why

A well-designed chunk strategy is crucial for optimal application performance:

1. **Maximizes Cache Efficiency**: By grouping dependencies by change frequency, you ensure that stable code (like React itself) stays cached while frequently updated code can be refreshed without re-downloading everything

2. **Balances HTTP Requests vs Size**: Too few chunks mean large downloads; too many chunks increase HTTP overhead. The sweet spot is typically 5-15 chunks for most applications

3. **Optimizes Critical Path**: Core chunks needed for initial render can be prioritized while feature-specific chunks (charts, editors) load on demand

4. **Improves Update Experience**: When you deploy updates, users only re-download the changed chunks, making subsequent visits fast

5. **Enables Parallel Loading**: Multiple smaller chunks can be downloaded in parallel, better utilizing available bandwidth

Chunk Strategy Guidelines:
- **Core chunks (< 50KB gzipped)**: React, React DOM, router
- **Feature chunks (50-150KB gzipped)**: State management, UI library
- **Large library chunks**: Isolate libraries > 100KB (charts, editors, PDF)
- **Application chunks**: Split by route or feature
- **Shared chunks**: Common components used across multiple routes


### 1.3 Configure Automatic Code Splitting

## Configure Automatic Code Splitting

**Impact: CRITICAL (Better caching, faster initial loads)**

Configure Vite to automatically split your application code into smaller chunks for better caching and faster initial loads.

## Bad Example

```tsx
// vite.config.ts - No code splitting configuration
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    // All code bundled into a single file
    rollupOptions: {
      output: {
        manualChunks: undefined,
      },
    },
  },
});
```

```tsx
// App.tsx - All imports at top level
import Dashboard from './pages/Dashboard';
import Settings from './pages/Settings';
import Profile from './pages/Profile';
import Analytics from './pages/Analytics';
import Reports from './pages/Reports';

function App() {
  return (
    <Routes>
      <Route path="/dashboard" element={<Dashboard />} />
      <Route path="/settings" element={<Settings />} />
      <Route path="/profile" element={<Profile />} />
      <Route path="/analytics" element={<Analytics />} />
      <Route path="/reports" element={<Reports />} />
    </Routes>
  );
}
```

## Good Example

```tsx
// vite.config.ts - Proper code splitting configuration
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // Vendor chunk for React ecosystem
          'react-vendor': ['react', 'react-dom', 'react-router-dom'],
          // UI library chunk
          'ui-vendor': ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu'],
          // Utility libraries chunk
          'utils-vendor': ['lodash-es', 'date-fns', 'zod'],
        },
      },
    },
    // Generate chunk size warnings
    chunkSizeWarningLimit: 500,
  },
});
```

```tsx
// App.tsx - Lazy loaded routes
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';
import { LoadingSpinner } from './components/LoadingSpinner';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));
const Profile = lazy(() => import('./pages/Profile'));
const Analytics = lazy(() => import('./pages/Analytics'));
const Reports = lazy(() => import('./pages/Reports'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
        <Route path="/profile" element={<Profile />} />
        <Route path="/analytics" element={<Analytics />} />
        <Route path="/reports" element={<Reports />} />
      </Routes>
    </Suspense>
  );
}
```

## Why

Code splitting is essential for modern web applications because:

1. **Faster Initial Load**: Users download only the code needed for the current page, reducing Time to Interactive (TTI)

2. **Better Caching**: Smaller, separate chunks can be cached independently. When you update one part of your app, users only need to re-download that specific chunk

3. **Parallel Downloads**: Browsers can download multiple smaller files simultaneously, utilizing available bandwidth more efficiently

4. **Reduced Memory Usage**: Loading code on-demand means less JavaScript needs to be parsed and executed upfront

5. **Improved Core Web Vitals**: Smaller initial bundles directly improve Largest Contentful Paint (LCP) and First Input Delay (FID) metrics

Vite's built-in Rollup configuration makes code splitting straightforward with `manualChunks` for vendor libraries and dynamic imports for route-based splitting.


### 1.4 Configure Build-Time Compression

## Configure Build-Time Compression

**Impact: HIGH (60-80% smaller asset size)**

Configure build-time compression to serve pre-compressed assets, reducing server load and improving delivery speed.

## Bad Example

```tsx
// vite.config.ts - No compression configured
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    // Relying only on server-side compression
    // which adds CPU overhead on every request
  },
});
```

```tsx
// server.ts - Runtime compression adds latency
import express from 'express';
import compression from 'compression';

const app = express();

// Compresses every response on-the-fly
// This adds latency and CPU usage
app.use(compression());
app.use(express.static('dist'));
```

## Good Example

```tsx
// vite.config.ts - Pre-compress assets during build
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import viteCompression from 'vite-plugin-compression';

export default defineConfig({
  plugins: [
    react(),
    // Generate gzip compressed files
    viteCompression({
      algorithm: 'gzip',
      ext: '.gz',
      threshold: 1024, // Only compress files > 1KB
      deleteOriginFile: false, // Keep original files
    }),
    // Also generate Brotli compressed files for modern browsers
    viteCompression({
      algorithm: 'brotliCompress',
      ext: '.br',
      threshold: 1024,
    }),
  ],
  build: {
    // Ensure assets are optimized before compression
    cssMinify: true,
    minify: 'esbuild',
  },
});
```

```tsx
// vite.config.ts - Advanced compression with custom options
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import viteCompression from 'vite-plugin-compression';
import { constants as zlibConstants } from 'zlib';

export default defineConfig({
  plugins: [
    react(),
    // Gzip with optimal settings
    viteCompression({
      algorithm: 'gzip',
      ext: '.gz',
      threshold: 1024,
      compressionOptions: {
        level: 9, // Maximum compression
      },
      filter: /\.(js|css|html|json|svg|txt|xml|wasm)$/i,
    }),
    // Brotli with optimal settings
    viteCompression({
      algorithm: 'brotliCompress',
      ext: '.br',
      threshold: 1024,
      compressionOptions: {
        params: {
          [zlibConstants.BROTLI_PARAM_QUALITY]: 11, // Maximum quality
        },
      },
      filter: /\.(js|css|html|json|svg|txt|xml|wasm)$/i,
    }),
  ],
});
```

```nginx
# nginx.conf - Serve pre-compressed files
server {
    listen 80;
    root /var/www/app/dist;

    # Enable gzip and brotli static serving
    gzip_static on;
    brotli_static on;

    location ~* \.(js|css|html|json|svg|txt|xml|wasm)$ {
        # Try to serve pre-compressed file first
        gzip_static on;
        brotli_static on;

        # Fallback to original if compressed version doesn't exist
        try_files $uri $uri/ =404;

        # Add proper cache headers
        add_header Cache-Control "public, max-age=31536000, immutable";
        add_header Vary "Accept-Encoding";
    }
}
```

```tsx
// express server with pre-compressed file serving
// server.ts
import express from 'express';
import expressStaticGzip from 'express-static-gzip';

const app = express();

// Serve pre-compressed files with proper content negotiation
app.use('/', expressStaticGzip('dist', {
  enableBrotli: true,
  orderPreference: ['br', 'gzip'], // Prefer Brotli over gzip
  serveStatic: {
    maxAge: '1y',
    immutable: true,
  },
}));

app.listen(3000);
```

## Why

Build-time compression provides significant benefits:

1. **Reduced Server CPU Usage**: Pre-compressed files eliminate the need for on-the-fly compression, freeing server resources for handling more requests

2. **Consistent Compression Quality**: Build-time compression can use maximum compression levels without impacting response latency

3. **Better Compression Ratios**: Higher compression levels achieve 10-20% better compression than real-time compression with reasonable latency

4. **Brotli Support**: Brotli offers 15-25% better compression than gzip, especially for text-based content

5. **Faster Time to First Byte**: No compression overhead means the server can start sending data immediately

Compression Comparison:
| Format | Browser Support | Typical Ratio | Best For |
|--------|-----------------|---------------|----------|
| Gzip | 95%+ | 70-80% | Universal fallback |
| Brotli | 90%+ | 80-90% | Modern browsers |

Best Practices:
- Generate both gzip and Brotli versions for maximum compatibility
- Set threshold to avoid compressing small files (overhead > benefit)
- Exclude already-compressed formats (images, videos, fonts)
- Configure server to serve pre-compressed files with proper Content-Encoding headers
- Use maximum compression levels during build (slower build, faster delivery)


### 1.5 Configure Manual Chunks for Vendor Separation

## Configure Manual Chunks for Vendor Separation

**Impact: CRITICAL (Optimal caching and parallel loading)**

Without manual chunks, Vite bundles all vendor dependencies into a single chunk or mixes them with application code. This leads to:
- Large initial bundle downloads
- Poor cache efficiency (vendor code changes with app code)
- Slower subsequent page loads

## Incorrect

```typescript
// vite.config.ts
export default defineConfig({
  plugins: [react()],
  build: {
    // No manual chunks configured
    // All code bundled together
  },
})
```

**Problem:** React, React DOM, and other vendors are bundled with your application code. When you update your app, users must re-download everything.

## Correct

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // Core React - rarely changes
          'vendor-react': ['react', 'react-dom'],

          // Router - changes occasionally
          'vendor-router': ['react-router-dom'],

          // UI library - if using one
          // 'vendor-ui': ['@headlessui/react', '@heroicons/react'],

          // State management
          // 'vendor-state': ['zustand', '@tanstack/react-query'],
        },
      },
    },
  },
})
```

## Advanced: Dynamic Manual Chunks

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks(id) {
          // Node modules go to vendor chunk
          if (id.includes('node_modules')) {
            // Split large libraries into separate chunks
            if (id.includes('react-dom')) {
              return 'vendor-react-dom'
            }
            if (id.includes('react')) {
              return 'vendor-react'
            }
            if (id.includes('@tanstack')) {
              return 'vendor-tanstack'
            }
            // Other node_modules
            return 'vendor'
          }
        },
      },
    },
  },
})
```

## Benefits

- **Better caching:** Vendor chunks cached separately from app code
- **Parallel loading:** Browser can download multiple chunks simultaneously
- **Smaller updates:** App changes don't invalidate vendor cache


### 1.6 Configure Optimal Minification Settings

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


### 1.7 Use Terser for Production Minification

## Use Terser for Production Minification

**Impact: CRITICAL (5-10% smaller bundles)**

Vite uses esbuild for minification by default (fast but less optimal). Terser produces smaller bundles through advanced optimizations like dead code elimination and console removal.

## Incorrect

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    // Default esbuild minification
    // No console removal
    // Less aggressive optimization
  },
})
```

## Correct

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  build: {
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,     // Remove console.log
        drop_debugger: true,    // Remove debugger
        pure_funcs: [           // Remove specific functions
          'console.info',
          'console.debug',
          'console.warn',
        ],
      },
      mangle: {
        safari10: true,         // Safari 10 compatibility
      },
      format: {
        comments: false,        // Remove all comments
      },
    },
  },
})
```

## Keep Error Logging

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: false,    // Keep console
        pure_funcs: [
          'console.log',        // Remove only log
          'console.debug',
          'console.info',
          // Keep console.error and console.warn for debugging
        ],
      },
    },
  },
})
```

## Development vs Production

```typescript
// vite.config.ts
export default defineConfig(({ mode }) => ({
  build: {
    minify: mode === 'production' ? 'terser' : 'esbuild',
    terserOptions: mode === 'production' ? {
      compress: {
        drop_console: true,
        drop_debugger: true,
      },
    } : undefined,
  },
}))
```

## Comparison

| Feature | esbuild | Terser |
|---------|---------|--------|
| Speed | Very Fast | Slower |
| Bundle Size | Good | Better (5-10% smaller) |
| Console Removal | Manual | Built-in |
| Dead Code | Basic | Advanced |

## When to Use Each

- **esbuild:** Development, fast builds, CI where speed matters
- **Terser:** Production builds, when bundle size is critical

## Impact

- 5-10% smaller production bundles
- No debug code in production
- Better dead code elimination


### 1.8 Configure Source Maps for Production Debugging

## Configure Source Maps for Production Debugging

**Impact: MEDIUM (Better error tracking without exposing source)**

Configure source maps appropriately for debugging in development and error tracking in production without exposing source code.

## Bad Example

```tsx
// vite.config.ts - Source maps disabled or misconfigured
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    // Source maps completely disabled - makes debugging production issues impossible
    sourcemap: false,
  },
});
```

```tsx
// Or exposing full source maps in production
export default defineConfig({
  plugins: [react()],
  build: {
    // Full source maps accessible to everyone
    sourcemap: true, // Creates .map files served publicly
  },
});
```

```tsx
// Deployment exposing source maps publicly
// server.ts
import express from 'express';

const app = express();

// BAD: Serves everything including .map files
app.use(express.static('dist'));
```

## Good Example

```tsx
// vite.config.ts - Environment-appropriate source map configuration
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig(({ mode }) => ({
  plugins: [react()],
  build: {
    // Use 'hidden' for production - generates maps but doesn't link them in bundles
    // Use 'true' for staging - full debugging capability
    sourcemap: mode === 'production' ? 'hidden' : true,
  },
}));
```

```tsx
// vite.config.ts - Advanced source map configuration
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig(({ mode }) => ({
  plugins: [react()],
  build: {
    sourcemap: mode === 'production' ? 'hidden' : true,
    rollupOptions: {
      output: {
        sourcemapExcludeSources: mode === 'production', // Exclude source content
      },
    },
  },
  css: {
    devSourcemap: true, // CSS source maps in development
  },
}));
```

```tsx
// Upload source maps to error tracking service
// scripts/upload-sourcemaps.ts
import { execSync } from 'child_process';

const SENTRY_AUTH_TOKEN = process.env.SENTRY_AUTH_TOKEN;
const SENTRY_ORG = process.env.SENTRY_ORG;
const SENTRY_PROJECT = process.env.SENTRY_PROJECT;
const RELEASE_VERSION = process.env.RELEASE_VERSION;

// Upload source maps to Sentry
execSync(`
  sentry-cli releases files ${RELEASE_VERSION} upload-sourcemaps ./dist \
    --auth-token ${SENTRY_AUTH_TOKEN} \
    --org ${SENTRY_ORG} \
    --project ${SENTRY_PROJECT} \
    --url-prefix '~/'
`);

// Delete source maps after upload (don't deploy them)
execSync('find ./dist -name "*.map" -delete');

console.log('Source maps uploaded and deleted from build');
```

```tsx
// vite.config.ts - Integration with Sentry plugin
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { sentryVitePlugin } from '@sentry/vite-plugin';

export default defineConfig(({ mode }) => ({
  plugins: [
    react(),
    mode === 'production' && sentryVitePlugin({
      org: process.env.SENTRY_ORG,
      project: process.env.SENTRY_PROJECT,
      authToken: process.env.SENTRY_AUTH_TOKEN,
      release: {
        name: process.env.RELEASE_VERSION,
      },
      sourcemaps: {
        assets: './dist/**',
        // Delete source maps after upload
        filesToDeleteAfterUpload: './dist/**/*.map',
      },
    }),
  ].filter(Boolean),
  build: {
    sourcemap: true, // Required for Sentry plugin
  },
}));
```

```nginx
# nginx.conf - Block access to source maps
server {
    listen 80;
    root /var/www/app/dist;

    # Block access to source maps
    location ~* \.map$ {
        # Only allow from internal IPs
        allow 10.0.0.0/8;
        allow 192.168.0.0/16;
        deny all;

        # Or return 404 entirely
        # return 404;
    }
}
```

## Why

Proper source map configuration is critical for both debugging and security:

1. **Production Debugging**: Source maps enable readable stack traces from minified code, making it possible to debug production errors

2. **Security**: Full source maps expose your original source code. Using 'hidden' source maps prevents this while still enabling error tracking

3. **Error Tracking Integration**: Services like Sentry can use uploaded source maps to provide detailed error reports with original file names and line numbers

4. **Development Experience**: Full source maps in development enable seamless debugging with browser DevTools

5. **Legal Protection**: Keeping source maps private protects your intellectual property

Source Map Options in Vite:
| Option | Description | Use Case |
|--------|-------------|----------|
| `false` | No source maps | Not recommended |
| `true` | Generates and links .map files | Development/Staging |
| `'inline'` | Embeds maps in bundles | Development only |
| `'hidden'` | Generates .map files without link | Production |

Best Practices:
- Use `hidden` source maps for production builds
- Upload source maps to error tracking services before deployment
- Delete source maps from production deployments
- Configure server to block public access to any remaining .map files
- Enable CSS source maps in development for easier styling debug


### 1.9 Target Modern Browsers for Smaller Bundles

## Target Modern Browsers for Smaller Bundles

**Impact: CRITICAL (10-15% smaller bundles)**

Vite defaults to a broad browser target for compatibility. Modern browsers support ES2020+ features natively. Targeting older browsers includes unnecessary polyfills and transpilation, increasing bundle size.

## Incorrect

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    // Default target includes older browsers
    // Results in larger bundles with polyfills
  },
})
```

Or explicitly targeting old browsers:

```typescript
export default defineConfig({
  build: {
    target: 'es2015', // Too old, includes many polyfills
  },
})
```

## Correct

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  build: {
    // Target modern browsers
    target: 'esnext',

    // Or be specific about browser versions
    // target: ['es2020', 'edge88', 'firefox78', 'chrome87', 'safari14'],
  },
})
```

## With Legacy Browser Support

If you need to support older browsers, use the legacy plugin:

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import legacy from '@vitejs/plugin-legacy'

export default defineConfig({
  plugins: [
    react(),
    legacy({
      targets: ['defaults', 'not IE 11'],
      // Modern chunks for modern browsers
      // Legacy chunks only loaded by old browsers
    }),
  ],
  build: {
    target: 'esnext', // Modern build
  },
})
```

## Target Options

| Target | Use Case |
|--------|----------|
| `esnext` | Latest features, smallest bundle |
| `es2022` | Good balance, wide support |
| `es2020` | Broader support, slightly larger |
| Custom array | Specific browser versions |

## Impact

Targeting `esnext` vs `es2015` can reduce bundle size by 10-30% depending on the codebase.


### 1.10 Configure Build for Effective Tree Shaking

## Configure Build for Effective Tree Shaking

**Impact: CRITICAL (15-30% smaller bundles)**

Configure your Vite build to effectively eliminate dead code through tree shaking, reducing bundle size significantly.

## Bad Example

```tsx
// utils/index.ts - Barrel export that prevents tree shaking
export * from './strings';
export * from './numbers';
export * from './dates';
export * from './arrays';
export * from './objects';

// Using namespace imports
import * as utils from './utils';

function Component() {
  // Only using one function but importing everything
  return <div>{utils.formatDate(new Date())}</div>;
}
```

```tsx
// Importing entire libraries
import _ from 'lodash';
import moment from 'moment';

function processData(items: Item[]) {
  // Using only 2 functions but importing entire library
  return _.uniqBy(items, 'id').map(item => ({
    ...item,
    date: moment(item.date).format('YYYY-MM-DD'),
  }));
}
```

```json
// package.json - Missing sideEffects field
{
  "name": "my-app",
  "version": "1.0.0",
  "main": "dist/index.js",
  "module": "dist/index.esm.js"
}
```

## Good Example

```tsx
// utils/index.ts - Named exports for better tree shaking
export { formatString, capitalize, truncate } from './strings';
export { formatNumber, clamp, round } from './numbers';
export { formatDate, parseDate, isValidDate } from './dates';
export { unique, groupBy, sortBy } from './arrays';
export { pick, omit, merge } from './objects';

// Direct named imports
import { formatDate } from './utils';

function Component() {
  return <div>{formatDate(new Date())}</div>;
}
```

```tsx
// Import only what you need from tree-shakeable libraries
import uniqBy from 'lodash-es/uniqBy';
import { format } from 'date-fns';

function processData(items: Item[]) {
  return uniqBy(items, 'id').map(item => ({
    ...item,
    date: format(new Date(item.date), 'yyyy-MM-dd'),
  }));
}
```

```json
// package.json - Proper sideEffects configuration
{
  "name": "my-app",
  "version": "1.0.0",
  "main": "dist/index.js",
  "module": "dist/index.esm.js",
  "sideEffects": [
    "*.css",
    "*.scss",
    "./src/polyfills.ts"
  ]
}
```

```tsx
// vite.config.ts - Optimize dependencies for tree shaking
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      treeshake: {
        moduleSideEffects: 'no-external',
        propertyReadSideEffects: false,
        tryCatchDeoptimization: false,
      },
    },
  },
  optimizeDeps: {
    include: ['lodash-es'],
  },
});
```

## Why

Tree shaking is a critical optimization technique that:

1. **Dramatically Reduces Bundle Size**: Unused exports are eliminated from the final bundle. A library might be 100KB but you only include the 5KB you actually use

2. **Improves Load Performance**: Smaller bundles mean faster downloads, especially on mobile networks

3. **Enables Modular Architecture**: You can organize code in feature-rich modules without worrying about bloating the bundle

4. **Works with ES Modules**: Tree shaking relies on static analysis of ES module imports/exports, which is why ESM-compatible libraries like `lodash-es` and `date-fns` are preferred

5. **Compounds with Code Splitting**: Combined with code splitting, tree shaking ensures each chunk contains only the code it needs

Key practices for effective tree shaking:
- Use ES modules (`import`/`export`) instead of CommonJS (`require`/`module.exports`)
- Prefer libraries that ship ES module builds
- Avoid namespace imports (`import * as`)
- Configure `sideEffects` in package.json to help bundlers identify pure modules
- Use named exports instead of default exports where possible


### 1.11 Strategic Vendor Code Splitting

## Strategic Vendor Code Splitting

**Impact: CRITICAL (Optimal cache utilization, faster builds)**

Separate vendor dependencies from application code to optimize caching and reduce rebuild times.

## Bad Example

```tsx
// vite.config.ts - All code in single bundle
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        // No vendor splitting - everything in one file
        manualChunks: undefined,
      },
    },
  },
});
// Result: main-abc123.js (500KB) - changes on every deployment
```

```tsx
// Or all vendors in one giant chunk
manualChunks: {
  vendor: [
    'react', 'react-dom', 'react-router-dom',
    'lodash', 'date-fns', 'axios',
    '@tanstack/react-query', 'zustand',
    'recharts', 'd3', 'monaco-editor',
    // ... 50+ more packages
  ],
}
// Result: vendor-xyz789.js (2MB) - changes when ANY dependency updates
```

## Good Example

```tsx
// vite.config.ts - Strategic vendor splitting
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // Framework core - very stable, rarely changes
          'vendor-react': [
            'react',
            'react-dom',
            'react/jsx-runtime',
          ],

          // Routing - stable, changes occasionally
          'vendor-router': [
            'react-router-dom',
            '@remix-run/router',
          ],

          // State management - moderate change frequency
          'vendor-state': [
            '@tanstack/react-query',
            'zustand',
          ],

          // UI primitives - changes with design system updates
          'vendor-ui': [
            '@radix-ui/react-dialog',
            '@radix-ui/react-dropdown-menu',
            '@radix-ui/react-tooltip',
            '@radix-ui/react-popover',
            'class-variance-authority',
            'clsx',
          ],

          // Utilities - very stable
          'vendor-utils': [
            'lodash-es',
            'date-fns',
            'zod',
          ],
        },
      },
    },
  },
});
```

```tsx
// vite.config.ts - Dynamic vendor splitting with size awareness
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

// Large dependencies that should be isolated
const LARGE_DEPS = new Set([
  'recharts',
  'd3',
  'monaco-editor',
  'pdfjs-dist',
  'xlsx',
  'three',
]);

// Stable core dependencies
const CORE_DEPS = new Set([
  'react',
  'react-dom',
  'react/jsx-runtime',
]);

// Routing dependencies
const ROUTER_DEPS = new Set([
  'react-router-dom',
  'react-router',
  '@remix-run/router',
]);

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: (id) => {
          if (!id.includes('node_modules')) {
            return; // Let app code be handled by code splitting
          }

          // Extract package name
          const match = id.match(/node_modules\/([^/]+)/);
          if (!match) return;

          const packageName = match[1];

          // Core React - most stable
          if (CORE_DEPS.has(packageName)) {
            return 'vendor-react';
          }

          // Router
          if (ROUTER_DEPS.has(packageName)) {
            return 'vendor-router';
          }

          // Large dependencies get their own chunks
          if (LARGE_DEPS.has(packageName)) {
            return `vendor-${packageName}`;
          }

          // Scoped packages grouped by scope
          if (packageName.startsWith('@')) {
            const scope = packageName.split('/')[0];
            return `vendor-${scope}`;
          }

          // Everything else
          return 'vendor-misc';
        },
      },
    },
  },
});
```

```tsx
// vite.config.ts - Production-optimized with dep analysis
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: (id, { getModuleInfo }) => {
          if (!id.includes('node_modules')) return;

          const module = getModuleInfo(id);
          if (!module) return;

          // Check how many other modules import this one
          const importers = module.importers.length;

          // Heavily imported modules go to common chunk
          if (importers > 10) {
            return 'vendor-common';
          }

          // Extract and return appropriate chunk
          const match = id.match(/node_modules\/(@?[^/]+)/);
          if (match) {
            const pkg = match[1].replace('@', '').replace('/', '-');

            // Group by update frequency
            if (['react', 'react-dom'].includes(pkg)) {
              return 'vendor-react';
            }

            return `vendor-${pkg}`;
          }
        },
      },
    },
    chunkSizeWarningLimit: 500,
  },
});
```

## Why

Strategic vendor splitting provides multiple benefits:

1. **Optimal Cache Utilization**: Stable dependencies like React stay cached for months while only application code needs re-downloading on deployments

2. **Faster CI/CD**: Unchanged vendor chunks don't need regeneration, speeding up builds

3. **Parallel Downloads**: Multiple smaller vendor chunks download faster than one large chunk due to HTTP/2 multiplexing

4. **Granular Invalidation**: When you update `lodash`, only that chunk invalidates, not your entire vendor bundle

5. **Better Loading Strategy**: Critical vendor chunks can be preloaded while optional ones (charts, editors) load on demand

Vendor Splitting Strategy:

| Chunk | Contents | Size Target | Cache Duration |
|-------|----------|-------------|----------------|
| vendor-react | React core | ~40KB | 6+ months |
| vendor-router | React Router | ~15KB | 3+ months |
| vendor-state | State management | ~20KB | 1-3 months |
| vendor-ui | UI components | ~50KB | Monthly |
| vendor-utils | Utilities | ~30KB | 6+ months |
| vendor-[large] | Heavy libs | Varies | As needed |

Best Practices:
- Group by update frequency, not functionality
- Isolate large dependencies (>100KB) into their own chunks
- Keep vendor-react separate - it rarely changes
- Use chunk size warnings to identify opportunities for splitting
- Monitor actual chunk sizes with bundle analyzer


---

## 2. Code Splitting

**Impact: CRITICAL**

### 2.1 Lazy Load Non-Critical Components

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


### 2.2 Use Dynamic Imports for Heavy Components

## Use Dynamic Imports for Heavy Components

**Impact: CRITICAL (30-50% reduction in initial bundle)**

Heavy components like charts, editors, and complex forms shouldn't be loaded until needed. Dynamic imports allow loading code on-demand, reducing initial bundle size.

## Incorrect

```typescript
// All heavy libraries loaded upfront
import { Chart } from 'chart.js'
import ReactQuill from 'react-quill'
import { PDFViewer } from '@react-pdf/renderer'
import MonacoEditor from '@monaco-editor/react'

function Dashboard() {
  const [showChart, setShowChart] = useState(false)

  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Chart</button>
      {showChart && <Chart data={data} />}
    </div>
  )
}
```

**Problem:** Chart.js, React Quill, PDF renderer, and Monaco are all loaded even if never used.

## Correct

```typescript
import { lazy, Suspense, useState } from 'react'

// Lazy load heavy components
const Chart = lazy(() => import('./components/Chart'))
const Editor = lazy(() => import('./components/Editor'))
const PDFViewer = lazy(() => import('./components/PDFViewer'))

function Dashboard() {
  const [showChart, setShowChart] = useState(false)
  const [showEditor, setShowEditor] = useState(false)

  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Chart</button>
      <button onClick={() => setShowEditor(true)}>Show Editor</button>

      {showChart && (
        <Suspense fallback={<ChartSkeleton />}>
          <Chart data={data} />
        </Suspense>
      )}

      {showEditor && (
        <Suspense fallback={<EditorSkeleton />}>
          <Editor />
        </Suspense>
      )}
    </div>
  )
}
```

## Conditional Dynamic Import

```typescript
// Load library only when feature is activated
async function exportToPDF() {
  // pdf-lib is only loaded when user clicks export
  const { PDFDocument } = await import('pdf-lib')

  const pdfDoc = await PDFDocument.create()
  // ... generate PDF
}

function ExportButton() {
  const [loading, setLoading] = useState(false)

  const handleExport = async () => {
    setLoading(true)
    await exportToPDF()
    setLoading(false)
  }

  return (
    <button onClick={handleExport} disabled={loading}>
      {loading ? 'Generating...' : 'Export PDF'}
    </button>
  )
}
```

## Preload on Interaction

```typescript
const HeavyModal = lazy(() => import('./HeavyModal'))

function ModalTrigger() {
  const [isOpen, setIsOpen] = useState(false)

  // Preload when user shows intent
  const preload = () => {
    import('./HeavyModal')
  }

  return (
    <>
      <button
        onClick={() => setIsOpen(true)}
        onMouseEnter={preload}
        onFocus={preload}
      >
        Open Settings
      </button>

      {isOpen && (
        <Suspense fallback={<ModalSkeleton />}>
          <HeavyModal onClose={() => setIsOpen(false)} />
        </Suspense>
      )}
    </>
  )
}
```

## Feature Flag Based Loading

```typescript
// Only load admin features for admin users
function App({ user }) {
  const AdminPanel = user.isAdmin
    ? lazy(() => import('./AdminPanel'))
    : null

  return (
    <div>
      <MainContent />
      {AdminPanel && (
        <Suspense fallback={<Loading />}>
          <AdminPanel />
        </Suspense>
      )}
    </div>
  )
}
```

## Heavy Library Examples

Libraries that should typically be dynamically imported:
- Chart libraries (Chart.js, Recharts, D3)
- Rich text editors (React Quill, TipTap, Slate)
- Code editors (Monaco, CodeMirror)
- PDF libraries (react-pdf, pdf-lib)
- Date pickers with locales
- Map libraries (Mapbox, Google Maps)
- Markdown renderers

## Impact

- Initial bundle can be 50%+ smaller
- Faster Time to Interactive
- Better user experience on slow connections


### 2.3 Implement Route-Based Code Splitting

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


### 2.4 Split Large Library Dependencies

## Split Large Library Dependencies

**Impact: CRITICAL (Better caching for third-party code)**

Configure Vite to split large third-party libraries into separate chunks for optimal caching and loading strategies.

## Bad Example

```tsx
// vite.config.ts - No library chunk configuration
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        // All libraries bundled together
        manualChunks: undefined,
      },
    },
  },
});
// Result: vendor.js contains react, lodash, moment, chart.js, monaco-editor (3MB+)
```

```tsx
// Or naive splitting that creates too many chunks
manualChunks: (id) => {
  if (id.includes('node_modules')) {
    // Every package gets its own chunk - creates hundreds of files
    return id.split('node_modules/')[1].split('/')[0];
  }
}
// Result: 100+ small HTTP requests, negating HTTP/2 benefits
```

## Good Example

```tsx
// vite.config.ts - Strategic library chunk splitting
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

// Define library groups by size and usage patterns
const FRAMEWORK_LIBS = ['react', 'react-dom', 'scheduler'];
const ROUTER_LIBS = ['react-router', 'react-router-dom', '@remix-run/router'];
const STATE_LIBS = ['zustand', '@tanstack/react-query', 'immer'];
const UI_LIBS = ['@radix-ui', '@headlessui', 'framer-motion'];
const FORM_LIBS = ['react-hook-form', '@hookform/resolvers', 'zod'];
const DATE_LIBS = ['date-fns', 'dayjs'];

// Large libraries that should be isolated
const LARGE_LIBS = ['monaco-editor', 'recharts', 'd3', 'three', 'pdfjs-dist'];

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: (id) => {
          if (!id.includes('node_modules')) return;

          // Framework core
          if (FRAMEWORK_LIBS.some(lib => id.includes(`/node_modules/${lib}/`))) {
            return 'lib-react';
          }

          // Router
          if (ROUTER_LIBS.some(lib => id.includes(`/node_modules/${lib}/`) ||
                                       id.includes(`/node_modules/${lib.replace('/', '-')}/`))) {
            return 'lib-router';
          }

          // State management
          if (STATE_LIBS.some(lib => id.includes(`/node_modules/${lib}/`))) {
            return 'lib-state';
          }

          // UI components
          if (UI_LIBS.some(lib => id.includes(`/node_modules/${lib}/`))) {
            return 'lib-ui';
          }

          // Forms
          if (FORM_LIBS.some(lib => id.includes(`/node_modules/${lib}/`))) {
            return 'lib-forms';
          }

          // Date utilities
          if (DATE_LIBS.some(lib => id.includes(`/node_modules/${lib}/`))) {
            return 'lib-dates';
          }

          // Large libraries get their own chunks
          for (const lib of LARGE_LIBS) {
            if (id.includes(`/node_modules/${lib}/`)) {
              return `lib-${lib}`;
            }
          }

          // Remaining node_modules
          return 'lib-vendor';
        },
      },
    },
  },
});
```

```tsx
// vite.config.ts - Size-aware automatic chunking
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: (id, { getModuleInfo }) => {
          if (!id.includes('node_modules')) return;

          const getPackageName = (id: string) => {
            const match = id.match(/node_modules\/(@[^/]+\/[^/]+|[^/]+)/);
            return match ? match[1] : null;
          };

          const packageName = getPackageName(id);
          if (!packageName) return;

          // Always isolate React
          if (['react', 'react-dom'].includes(packageName)) {
            return 'lib-react';
          }

          // Get module info for analysis
          const moduleInfo = getModuleInfo(id);

          // Large modules or heavily imported modules
          if (moduleInfo) {
            const importedCount = moduleInfo.importedIds?.length || 0;

            // Modules with many imports are likely large
            if (importedCount > 50) {
              return `lib-${packageName.replace('@', '').replace('/', '-')}`;
            }
          }

          // Group scoped packages
          if (packageName.startsWith('@')) {
            const scope = packageName.split('/')[0].replace('@', '');

            // Well-known UI scopes
            if (['radix-ui', 'headlessui'].includes(scope)) {
              return 'lib-ui';
            }

            // Tanstack packages
            if (scope === 'tanstack') {
              return 'lib-tanstack';
            }

            return `lib-${scope}`;
          }

          return 'lib-vendor';
        },
      },
    },
    chunkSizeWarningLimit: 300, // Warn for chunks > 300KB
  },
});
```

```tsx
// Package.json scripts for chunk analysis
{
  "scripts": {
    "build": "vite build",
    "build:analyze": "vite build && npx vite-bundle-visualizer",
    "analyze": "npx source-map-explorer dist/assets/*.js"
  }
}

// After running build:analyze, review chunk sizes:
// lib-react.js     ~40KB gzipped (stable)
// lib-router.js    ~12KB gzipped (stable)
// lib-state.js     ~15KB gzipped (moderate changes)
// lib-ui.js        ~45KB gzipped (changes with design)
// lib-forms.js     ~25KB gzipped (stable)
// lib-vendor.js    ~30KB gzipped (catch-all)
// lib-recharts.js  ~80KB gzipped (lazy loaded)
// lib-monaco.js    ~500KB gzipped (lazy loaded)
```

## Why

Strategic library chunking is essential for production performance:

1. **Optimal Cache Granularity**: When you update lodash, users don't need to re-download React. Each library group can be cached independently

2. **Predictable Bundle Sizes**: Grouping by purpose creates consistent chunk sizes, making performance budgets easier to maintain

3. **Loading Prioritization**: Critical libraries (React) can load first while heavy optional libraries (charts) load on demand

4. **HTTP/2 Efficiency**: Modern HTTP/2 handles multiple small requests efficiently, but 5-15 chunks is more optimal than 100+

5. **Build Performance**: Rollup can parallelize chunk generation, speeding up CI/CD pipelines

Library Chunk Strategy:

| Chunk | Contents | Size Target | Priority |
|-------|----------|-------------|----------|
| lib-react | React core | ~40KB | Critical |
| lib-router | Routing | ~15KB | Critical |
| lib-state | State management | ~20KB | High |
| lib-ui | UI primitives | ~50KB | High |
| lib-forms | Form handling | ~25KB | Medium |
| lib-dates | Date utilities | ~10KB | Low |
| lib-vendor | Miscellaneous | <50KB | Low |
| lib-[heavy] | Monaco, D3, etc | Varies | Lazy |

Best Practices:
- Keep critical path libraries in small, separate chunks
- Isolate large libraries (>100KB) that can be lazy loaded
- Group libraries by update frequency for better caching
- Use bundle analyzer to verify chunk sizes
- Set chunk size warnings to catch regressions


### 2.5 Prefetch Code Chunks on User Intent

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


### 2.6 Use React.lazy() for Route-Based Splitting

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


### 2.7 Strategic Suspense Boundaries for Lazy Loading

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


---

## 3. Development Performance

**Impact: HIGH**

### 3.1 Configure Dependency Pre-bundling

## Configure Dependency Pre-bundling

**Impact: HIGH (2-5× faster cold start)**

Vite pre-bundles dependencies to convert CommonJS/UMD to ESM and reduce the number of module requests. Proper configuration speeds up cold starts and prevents runtime issues.

## Incorrect

```typescript
// vite.config.ts
export default defineConfig({
  // No optimizeDeps configuration
  // Vite auto-detects but may miss some deps
})
```

**Problems:**
- Some dependencies may not be pre-bundled
- Cold start can be slow with many dependencies
- Runtime errors from unbundled CommonJS modules

## Correct

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],

  optimizeDeps: {
    // Explicitly include dependencies that should be pre-bundled
    include: [
      'react',
      'react-dom',
      'react-router-dom',
      '@tanstack/react-query',
      'zustand',
      'axios',
      'date-fns',
      // Include nested dependencies if needed
      'react-dom/client',
    ],

    // Exclude dependencies that don't need pre-bundling
    // (already ESM or causing issues)
    exclude: [
      // '@some/esm-only-package',
    ],
  },
})
```

## Force Re-bundling

```typescript
// vite.config.ts
export default defineConfig({
  optimizeDeps: {
    // Force re-bundling (useful when deps update)
    force: true, // Remove after resolving issues
  },
})
```

Or via CLI:
```bash
vite --force
# or
vite optimize --force
```

## Handle CommonJS Dependencies

```typescript
// vite.config.ts
export default defineConfig({
  optimizeDeps: {
    include: [
      // Some packages have deeply nested CommonJS
      'lodash-es',
      // Force include linked packages
      'linked-package > some-dep',
    ],

    // ESBuild options for dependency optimization
    esbuildOptions: {
      // Handle packages that use Node.js globals
      define: {
        global: 'globalThis',
      },
    },
  },
})
```

## Warmup Frequently Used Files

```typescript
// vite.config.ts (Vite 5+)
export default defineConfig({
  server: {
    warmup: {
      // Pre-transform these files on server start
      clientFiles: [
        './src/main.tsx',
        './src/App.tsx',
        './src/components/index.ts',
      ],
    },
  },
})
```

## Debug Pre-bundling

```bash
# See what's being pre-bundled
DEBUG=vite:deps vite

# Check the pre-bundle output
ls node_modules/.vite/deps/
```

## Impact

- 2-5x faster cold start
- Eliminates "optimizing dependencies" during development
- Prevents CommonJS/ESM compatibility issues


### 3.2 Structure Components for Fast Refresh

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


### 3.3 Configure HMR for Optimal Development

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


---

## 4. Asset Handling

**Impact: HIGH**

### 4.1 Optimize Image Loading and Format

## Optimize Image Loading and Format

**Impact: HIGH (40-70% reduction in image payload)**

Unoptimized images are often the largest assets, significantly impacting page load time. Proper image handling reduces bandwidth and improves Core Web Vitals.

## Incorrect

```typescript
// Large images loaded eagerly
function Gallery() {
  return (
    <div>
      <img src="/images/hero.png" />
      <img src="/images/feature1.png" />
      <img src="/images/feature2.png" />
      <img src="/images/feature3.png" />
    </div>
  )
}
```

**Problems:**
- No lazy loading
- No responsive images
- No explicit dimensions (layout shift)
- Potentially oversized images

## Correct

```typescript
function Gallery() {
  return (
    <div>
      {/* Critical above-fold image */}
      <img
        src="/images/hero.webp"
        alt="Hero banner"
        width={1200}
        height={600}
        fetchPriority="high"
      />

      {/* Below-fold images - lazy load */}
      <img
        src="/images/feature1.webp"
        alt="Feature 1"
        width={400}
        height={300}
        loading="lazy"
        decoding="async"
      />
      <img
        src="/images/feature2.webp"
        alt="Feature 2"
        width={400}
        height={300}
        loading="lazy"
        decoding="async"
      />
    </div>
  )
}
```

## Responsive Images

```typescript
function ResponsiveImage() {
  return (
    <picture>
      {/* WebP for modern browsers */}
      <source
        srcSet="/images/hero-480.webp 480w,
                /images/hero-768.webp 768w,
                /images/hero-1200.webp 1200w"
        type="image/webp"
        sizes="(max-width: 480px) 480px,
               (max-width: 768px) 768px,
               1200px"
      />
      {/* Fallback for older browsers */}
      <img
        src="/images/hero-1200.jpg"
        alt="Hero image"
        width={1200}
        height={600}
        loading="lazy"
      />
    </picture>
  )
}
```

## Vite Image Optimization Plugin

```bash
npm install vite-plugin-image-optimizer -D
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import { ViteImageOptimizer } from 'vite-plugin-image-optimizer'

export default defineConfig({
  plugins: [
    ViteImageOptimizer({
      png: {
        quality: 80,
      },
      jpeg: {
        quality: 80,
      },
      webp: {
        lossless: true,
      },
    }),
  ],
})
```

## Image Component Pattern

```typescript
// components/Image.tsx
interface ImageProps {
  src: string
  alt: string
  width: number
  height: number
  priority?: boolean
  className?: string
}

export function Image({
  src,
  alt,
  width,
  height,
  priority = false,
  className,
}: ImageProps) {
  return (
    <img
      src={src}
      alt={alt}
      width={width}
      height={height}
      loading={priority ? 'eager' : 'lazy'}
      decoding={priority ? 'sync' : 'async'}
      fetchPriority={priority ? 'high' : 'auto'}
      className={className}
    />
  )
}

// Usage
<Image
  src="/hero.webp"
  alt="Hero"
  width={1200}
  height={600}
  priority
/>
```

## Inline Small Images

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    // Inline images smaller than 4kb as base64
    assetsInlineLimit: 4096,
  },
})
```

## Background Images

```typescript
// For CSS background images, use ?url suffix
import heroImage from './images/hero.webp?url'

function Hero() {
  return (
    <div
      className="hero"
      style={{ backgroundImage: `url(${heroImage})` }}
    />
  )
}
```

## Impact

- 40-70% reduction in image payload
- Better LCP (Largest Contentful Paint)
- Reduced CLS (Cumulative Layout Shift)


### 4.2 Use SVGs as React Components

## Use SVGs as React Components

**Impact: HIGH (Better styling and integration)**

SVGs can be used as images or as React components. Using them as components enables styling with CSS, dynamic colors, and better integration with React.

## Incorrect

```typescript
// Using SVG as image - limited styling options
function Logo() {
  return <img src="/logo.svg" alt="Logo" className="w-8 h-8" />
}

// Inline SVG everywhere - duplicated code
function Icon() {
  return (
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
      <path d="M12 2L2 7l10 5 10-5-10-5zM2 17l10 5 10-5M2 12l10 5 10-5" />
    </svg>
  )
}
```

## Correct

Install vite-plugin-svgr:

```bash
npm install vite-plugin-svgr -D
```

Configure Vite:

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import svgr from 'vite-plugin-svgr'

export default defineConfig({
  plugins: [
    react(),
    svgr({
      // Export as React component by default
      exportAsDefault: false,
      // SVG options
      svgrOptions: {
        plugins: ['@svgr/plugin-svgo', '@svgr/plugin-jsx'],
        svgoConfig: {
          plugins: [
            {
              name: 'removeViewBox',
              active: false, // Keep viewBox for scaling
            },
          ],
        },
      },
    }),
  ],
})
```

Usage:

```typescript
// Import as React component
import { ReactComponent as Logo } from './assets/logo.svg'
// Or with default export config
import Logo from './assets/logo.svg?react'

// Import as URL (for img src)
import logoUrl from './assets/logo.svg'

function Header() {
  return (
    <header>
      {/* As component - fully styleable */}
      <Logo className="w-8 h-8 text-blue-600 hover:text-blue-700" />

      {/* As image */}
      <img src={logoUrl} alt="Logo" className="w-8 h-8" />
    </header>
  )
}
```

## TypeScript Support

```typescript
// src/vite-env.d.ts
/// <reference types="vite/client" />
/// <reference types="vite-plugin-svgr/client" />

// Or manually declare
declare module '*.svg?react' {
  import type { FunctionComponent, SVGProps } from 'react'
  const content: FunctionComponent<SVGProps<SVGSVGElement>>
  export default content
}

declare module '*.svg' {
  const content: string
  export default content
}
```

## Dynamic SVG Colors

```typescript
// SVG component inherits currentColor
import SearchIcon from './assets/search.svg?react'

function SearchButton({ active }: { active: boolean }) {
  return (
    <button className={active ? 'text-blue-600' : 'text-gray-400'}>
      {/* Icon color follows text color */}
      <SearchIcon className="w-5 h-5" />
      Search
    </button>
  )
}
```

## Icon Component Pattern

```typescript
// components/Icon.tsx
import type { SVGProps, FunctionComponent } from 'react'

// Import all icons
import HomeIcon from '@/assets/icons/home.svg?react'
import SettingsIcon from '@/assets/icons/settings.svg?react'
import UserIcon from '@/assets/icons/user.svg?react'

const icons = {
  home: HomeIcon,
  settings: SettingsIcon,
  user: UserIcon,
} as const

type IconName = keyof typeof icons

interface IconProps extends SVGProps<SVGSVGElement> {
  name: IconName
  size?: number
}

export function Icon({ name, size = 24, className, ...props }: IconProps) {
  const IconComponent = icons[name]
  return (
    <IconComponent
      width={size}
      height={size}
      className={className}
      {...props}
    />
  )
}

// Usage
<Icon name="home" size={20} className="text-gray-600" />
```

## Impact

- SVGs styleable with Tailwind/CSS
- Dynamic colors without multiple SVG files
- Better tree-shaking of unused icons


---

## 5. Environment Configuration

**Impact: MEDIUM**

### 5.1 Use VITE_ Prefix for Environment Variables

## Use VITE_ Prefix for Environment Variables

**Impact: MEDIUM (Security and proper configuration)**

Vite only exposes environment variables prefixed with `VITE_` to client-side code. This prevents accidental exposure of sensitive server-side variables.

## Incorrect

```env
# .env
API_KEY=secret123
DATABASE_URL=postgres://...
APP_TITLE=My App
```

```typescript
// This won't work - variables not exposed
const apiKey = import.meta.env.API_KEY // undefined
const title = import.meta.env.APP_TITLE // undefined
```

**Problem:** Variables without `VITE_` prefix are not available in client code.

## Correct

```env
# .env
# Client-side variables (exposed to browser)
VITE_API_URL=https://api.example.com
VITE_APP_TITLE=My App
VITE_ENABLE_ANALYTICS=true

# Server-side only (NOT exposed to browser)
DATABASE_URL=postgres://...
API_SECRET=secret123
```

```typescript
// Access client-side variables
const apiUrl = import.meta.env.VITE_API_URL
const appTitle = import.meta.env.VITE_APP_TITLE
const enableAnalytics = import.meta.env.VITE_ENABLE_ANALYTICS === 'true'

// Built-in variables
const isDev = import.meta.env.DEV
const isProd = import.meta.env.PROD
const mode = import.meta.env.MODE
const baseUrl = import.meta.env.BASE_URL
```

## Type-Safe Environment Variables

```typescript
// src/vite-env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_URL: string
  readonly VITE_APP_TITLE: string
  readonly VITE_ENABLE_ANALYTICS: string
  // Add more as needed
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

## Environment Files

```
.env                # Loaded in all cases
.env.local          # Loaded in all cases, ignored by git
.env.[mode]         # Only loaded in specified mode
.env.[mode].local   # Only loaded in specified mode, ignored by git
```

```env
# .env.development
VITE_API_URL=http://localhost:8000/api

# .env.production
VITE_API_URL=https://api.example.com

# .env.staging
VITE_API_URL=https://staging-api.example.com
```

## Runtime Configuration

For values that need to change without rebuild:

```typescript
// public/config.js (loaded at runtime)
window.APP_CONFIG = {
  apiUrl: 'https://api.example.com',
}

// src/config.ts
export const config = {
  apiUrl: window.APP_CONFIG?.apiUrl || import.meta.env.VITE_API_URL,
}
```

```html
<!-- index.html -->
<script src="/config.js"></script>
```

## Never Expose

```env
# WRONG - These should NEVER have VITE_ prefix
VITE_DATABASE_URL=...     # Server-only
VITE_API_SECRET=...       # Server-only
VITE_PRIVATE_KEY=...      # Server-only

# RIGHT - Keep sensitive data without prefix
DATABASE_URL=...
API_SECRET=...
PRIVATE_KEY=...
```

## Impact

- Prevents accidental exposure of secrets
- Clear separation of client/server config
- Type safety catches undefined variables


---

## 6. HMR Optimization

**Impact: MEDIUM**

### 6.1 Preserve Component State with Fast Refresh

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


---

## 7. Bundle Analysis

**Impact: LOW-MEDIUM**

### 7.1 Ensure Proper Tree Shaking with ESM Imports

## Ensure Proper Tree Shaking with ESM Imports

**Impact: MEDIUM (20-40% smaller bundle with proper imports)**

Tree shaking removes unused code from bundles. Improper imports can prevent tree shaking, including entire libraries when only small parts are used.

## Incorrect

```typescript
// ❌ Imports entire library
import _ from 'lodash'
const result = _.get(obj, 'path')

// ❌ Namespace import prevents tree shaking
import * as utils from './utils'
utils.formatDate(date)

// ❌ Importing from barrel file
import { Button } from '@/components'
// If components/index.ts exports 50 components,
// all may be included
```

## Correct

```typescript
// ✅ Import only what you need
import get from 'lodash/get'
const result = get(obj, 'path')

// ✅ Or use lodash-es for better tree shaking
import { get } from 'lodash-es'

// ✅ Named imports allow tree shaking
import { formatDate } from './utils'
formatDate(date)

// ✅ Direct imports from source
import { Button } from '@/components/Button'
```

## Avoid Barrel Files for Large Libraries

```typescript
// ❌ components/index.ts (barrel file)
export * from './Button'
export * from './Input'
export * from './Modal'
export * from './Table'
export * from './Chart'
// ... 50 more components

// ❌ Consumer imports one, gets all
import { Button } from '@/components'
```

```typescript
// ✅ Direct imports
import { Button } from '@/components/Button'
import { Input } from '@/components/Input'

// ✅ Or use a smaller barrel for related components
// components/forms/index.ts
export { Input } from './Input'
export { Select } from './Select'
export { Checkbox } from './Checkbox'
```

## Check Tree Shaking with Visualizer

```bash
npm install rollup-plugin-visualizer -D
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import { visualizer } from 'rollup-plugin-visualizer'

export default defineConfig({
  plugins: [
    visualizer({
      filename: 'dist/stats.html',
      open: true,
      gzipSize: true,
    }),
  ],
})
```

## Side Effects Configuration

```json
// package.json
{
  "sideEffects": false
}

// Or specify files with side effects
{
  "sideEffects": [
    "*.css",
    "*.scss",
    "./src/polyfills.ts"
  ]
}
```

## Common Tree Shaking Issues

```typescript
// ❌ Default exports can be harder to tree shake
export default {
  formatDate,
  formatCurrency,
  formatNumber,
}

// ✅ Named exports tree shake better
export { formatDate }
export { formatCurrency }
export { formatNumber }
```

```typescript
// ❌ Re-exporting without type annotation
export { User } from './types'

// ✅ Type-only exports are removed
export type { User } from './types'
```

## Vite Configuration

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      // Treat these as external (not bundled)
      external: ['react', 'react-dom'],

      // Tree shake properly
      treeshake: {
        moduleSideEffects: false,
        propertyReadSideEffects: false,
      },
    },
  },
})
```

## Impact

- 10-50% smaller bundles depending on imports
- Faster load times
- Better caching (smaller chunks change less)


---

## 8. Advanced Patterns

**Impact: LOW**

No rules defined for this category yet.

---
