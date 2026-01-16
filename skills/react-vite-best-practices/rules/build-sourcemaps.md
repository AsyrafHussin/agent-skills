---
title: Configure Source Maps for Production Debugging
impact: MEDIUM
impactDescription: Better error tracking without exposing source
tags: build, sourcemaps, debugging, production, vite
---

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
