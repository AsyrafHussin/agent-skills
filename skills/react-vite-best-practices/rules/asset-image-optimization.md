---
title: Optimize Image Loading and Format
impact: HIGH
impactDescription: 40-70% reduction in image payload
tags: asset, images, optimization, webp, lazy-loading
---

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
