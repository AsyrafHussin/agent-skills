---
title: Configure Asset Hashing for Cache Busting
impact: CRITICAL
impactDescription: Ensures latest version delivery
tags: build, hashing, caching, assets, vite
---

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
