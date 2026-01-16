---
title: Use VITE_ Prefix for Environment Variables
impact: MEDIUM
impactDescription: Security and proper configuration
tags: env, configuration, security, vite, environment-variables
---

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
