# React + Vite Best Practices

Performance optimization guidelines for React applications built with Vite.

## Overview

This skill provides comprehensive guidance for:
- Vite configuration optimization
- Build output optimization
- Code splitting strategies
- Development experience improvements
- Asset handling best practices

## Categories

### 1. Build Optimization (Critical)
Configure Vite's build process for optimal production output.

### 2. Code Splitting (Critical)
Implement effective code splitting with React.lazy and dynamic imports.

### 3. Development Performance (High)
Optimize the development server and HMR for faster iteration.

### 4. Asset Handling (High)
Handle images, fonts, and other assets efficiently.

### 5. Environment Configuration (Medium)
Manage environment variables securely and type-safely.

### 6. HMR Optimization (Medium)
Ensure Hot Module Replacement works smoothly.

### 7. Bundle Analysis (Low-Medium)
Analyze and optimize bundle composition.

### 8. Advanced Patterns (Low)
SSR, library mode, and other advanced configurations.

## Quick Start

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  build: {
    target: 'esnext',
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
        },
      },
    },
  },
})
```

## Usage

This skill triggers automatically when:
- Configuring Vite projects
- Optimizing React performance
- Setting up code splitting
- Reviewing build output

## References

- [Vite Documentation](https://vitejs.dev/)
- [React Documentation](https://react.dev/)
- [Rollup Output Options](https://rollupjs.org/configuration-options/)
