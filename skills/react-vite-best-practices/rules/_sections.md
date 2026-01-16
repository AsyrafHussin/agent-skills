# Rule Sections

## Priority Levels

| Level | Description | When to Apply |
|-------|-------------|---------------|
| CRITICAL | Essential for production apps | Always |
| HIGH | Significant performance impact | Most projects |
| MEDIUM | Noticeable improvements | When optimizing |
| LOW | Minor optimizations | Large-scale apps |

## Section Overview

### Build Optimization (CRITICAL)
Rules for configuring Vite's production build. These have the highest impact on bundle size and load time.

### Code Splitting (CRITICAL)
Rules for splitting code effectively. Directly impacts initial load time and time-to-interactive.

### Development Performance (HIGH)
Rules for faster development iteration. Important for developer experience but doesn't affect production.

### Asset Handling (HIGH)
Rules for handling static assets. Affects both bundle size and runtime performance.

### Environment Configuration (MEDIUM)
Rules for managing environment variables. Important for security and configuration management.

### HMR Optimization (MEDIUM)
Rules for Hot Module Replacement. Affects development experience.

### Bundle Analysis (LOW-MEDIUM)
Rules for analyzing and understanding bundle composition. Useful for identifying optimization opportunities.

### Advanced Patterns (LOW)
Rules for specialized use cases like SSR, library mode, etc.
