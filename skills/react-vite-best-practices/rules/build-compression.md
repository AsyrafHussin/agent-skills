---
title: Configure Build-Time Compression
impact: HIGH
impactDescription: 60-80% smaller asset size
tags: build, compression, gzip, brotli, optimization
---

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
