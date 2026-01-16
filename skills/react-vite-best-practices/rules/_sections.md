# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. Build Optimization (build)

**Impact:** CRITICAL
**Description:** Optimizing Vite build configuration and output has the highest impact on production bundle size, load time, and caching efficiency. These rules ensure minimal bundle size and optimal delivery.

## 2. Code Splitting (split)

**Impact:** CRITICAL
**Description:** Strategic code splitting reduces initial bundle size by 50-80%, dramatically improving Time to Interactive and First Contentful Paint. Load only what's needed, when it's needed.

## 3. Development Performance (dev)

**Impact:** HIGH
**Description:** Fast development iteration with instant feedback loops. Optimizing Vite's dev server, dependency pre-bundling, and HMR reduces build times and improves developer productivity.

## 4. Asset Handling (asset)

**Impact:** HIGH
**Description:** Proper asset optimization (images, SVGs, fonts) reduces page weight by 40-70% and improves Core Web Vitals. Critical for performance on slower networks.

## 5. Environment Configuration (env)

**Impact:** MEDIUM
**Description:** Proper environment variable handling ensures security (no leaked secrets) and correct configuration across development, staging, and production environments.

## 6. HMR Optimization (hmr)

**Impact:** MEDIUM
**Description:** React Fast Refresh and HMR optimization preserve component state during development, enabling instant visual feedback without full page reloads.

## 7. Bundle Analysis (bundle)

**Impact:** LOW-MEDIUM
**Description:** Tools and techniques for analyzing bundle composition, identifying bloat, and ensuring proper tree shaking. Essential for maintaining long-term bundle health.

## 8. Advanced Patterns (advanced)

**Impact:** LOW
**Description:** Specialized patterns for SSR, library mode, multi-page apps, Web Workers, and WebAssembly. Apply only when these specific use cases are required.
