---
name: seo-best-practices
description: SEO patterns and conventions for web applications. Use when implementing meta tags, structured data, Core Web Vitals, sitemaps, Open Graph, or optimizing pages for search engines. Triggers on tasks involving SEO, search optimization, schema markup, or social sharing meta tags.
license: MIT
metadata:
  author: agent-skills
  version: "1.0.0"
---

# SEO Best Practices

Comprehensive SEO patterns for web applications built with React and Laravel. Contains 28 rules across 8 categories covering Core Web Vitals, technical SEO, structured data, performance, social sharing, and mobile-first indexing.

## Metadata

- **Version:** 1.0.0
- **Scope:** Framework-agnostic with React and Laravel examples
- **Rule Count:** 28 rules across 8 categories
- **License:** MIT

## When to Apply

Reference these guidelines when:
- Adding meta tags (`<title>`, description, canonical, robots)
- Implementing structured data (JSON-LD / Schema.org)
- Optimizing Core Web Vitals (LCP, INP, CLS)
- Setting up Open Graph and Twitter Card meta tags
- Creating sitemaps and robots.txt
- Building SEO-friendly React SPAs
- Configuring Laravel SEO middleware and packages
- Optimizing images, fonts, and loading performance for search

## Step 1: Detect Project Type

**Always check the project stack before giving advice.** Different stacks need different SEO approaches.

Check `package.json` and project structure:

| Signal | Project Type |
|--------|-------------|
| `@inertiajs/react` in dependencies | Laravel + Inertia + React |
| `resources/views/**/*.blade.php` only (no React) | Laravel Blade (server-rendered) |

**If Laravel Blade:** Apply `tech-`, `onpage-`, `schema-`, `perf-`, `social-`, `mobile-` rules. Meta tags go in Blade layouts. Sitemaps via `spatie/laravel-sitemap`. Skip `spa-` rules — pages are already server-rendered.

**If Laravel + Inertia + React:** Apply all rules. Meta tags via `@inertiaHead` in Blade layout + `<Head>` component from `@inertiajs/react` in React pages. For SSR, create `resources/js/ssr.jsx` using `createServer` from `@inertiajs/react/server`, add `ssr: 'resources/js/ssr.jsx'` to Vite config, build with `vite build && vite build --ssr`, and run `php artisan inertia:start-ssr`. Use `head-key` attribute on meta tags to prevent duplicates between layout and page. Focus on `schema-`, `social-`, and `perf-` rules.

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Core Web Vitals | CRITICAL | `cwv-` |
| 2 | Technical SEO | CRITICAL | `tech-` |
| 3 | On-Page SEO | HIGH | `onpage-` |
| 4 | Structured Data | HIGH | `schema-` |
| 5 | Performance SEO | HIGH | `perf-` |
| 6 | Social Sharing | HIGH | `social-` |
| 7 | React/SPA SEO | HIGH | `spa-` |
| 8 | Mobile-First | MEDIUM | `mobile-` |

## Quick Reference

### 1. Core Web Vitals (CRITICAL)

- `cwv-lcp` - Largest Contentful Paint optimization (<2.5s)
- `cwv-inp` - Interaction to Next Paint optimization (<200ms)
- `cwv-cls` - Cumulative Layout Shift prevention (<0.1)

### 2. Technical SEO (CRITICAL)

- `tech-meta-tags` - Essential HTML meta tags (title, description, canonical)
- `tech-canonical-urls` - Canonical URL implementation
- `tech-robots-txt` - Robots.txt configuration
- `tech-sitemap-xml` - XML sitemap generation
- `tech-url-structure` - SEO-friendly URL patterns

### 3. On-Page SEO (HIGH)

- `onpage-headings` - Heading hierarchy and structure
- `onpage-semantic-html` - Semantic HTML for SEO and accessibility
- `onpage-internal-linking` - Internal linking strategy
- `onpage-images` - Image optimization and alt text

### 4. Structured Data (HIGH)

- `schema-json-ld` - JSON-LD structured data basics
- `schema-article` - Article and BlogPosting markup
- `schema-product` - Product schema for e-commerce
- `schema-breadcrumb` - BreadcrumbList navigation markup

### 5. Performance SEO (HIGH)

- `perf-image-formats` - Modern image formats (WebP/AVIF)
- `perf-lazy-loading` - Lazy loading implementation
- `perf-font-loading` - Web font loading strategy
- `perf-resource-hints` - Preconnect, preload, and prefetch

### 6. Social Sharing (HIGH)

- `social-open-graph` - Open Graph meta tags
- `social-twitter-cards` - Twitter/X Card meta tags

### 7. React/SPA SEO (HIGH)

- `spa-rendering-strategy` - SSR vs CSR vs SSG for SEO
- `spa-meta-management` - Dynamic meta tag management
- `spa-routing` - SPA routing and crawlability

### 8. Mobile-First (MEDIUM)

- `mobile-viewport` - Viewport and responsive configuration
- `mobile-content-parity` - Content parity between mobile and desktop
- `mobile-ux` - Mobile UX requirements for SEO

## Essential Patterns

### Meta Tags (HTML)

```html
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Page Title — Site Name</title>
  <meta name="description" content="Concise 150-160 character description of page content." />
  <link rel="canonical" href="https://example.com/page" />

  <!-- Open Graph -->
  <meta property="og:title" content="Page Title" />
  <meta property="og:description" content="Description for social sharing." />
  <meta property="og:image" content="https://example.com/og-image.jpg" />
  <meta property="og:url" content="https://example.com/page" />
  <meta property="og:type" content="website" />

  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image" />
</head>
```

### JSON-LD Structured Data

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Article Title",
  "author": {
    "@type": "Person",
    "name": "Author Name"
  },
  "datePublished": "2026-03-14",
  "dateModified": "2026-03-14",
  "image": "https://example.com/article-image.jpg",
  "publisher": {
    "@type": "Organization",
    "name": "Site Name",
    "logo": {
      "@type": "ImageObject",
      "url": "https://example.com/logo.png"
    }
  }
}
</script>
```

### React Meta Management (Inertia.js)

```tsx
import { Head } from '@inertiajs/react'

interface Post {
  title: string;
  slug: string;
  excerpt: string;
  image: string;
}

function BlogPost({ post }: { post: Post }) {
  return (
    <>
      <Head>
        <title>{post.title} — Blog</title>
        <meta head-key="description" name="description" content={post.excerpt} />
        <link rel="canonical" href={`https://example.com/blog/${post.slug}`} />
        <meta property="og:title" content={post.title} />
        <meta property="og:description" content={post.excerpt} />
        <meta property="og:image" content={post.image} />
        <meta property="og:type" content="article" />
        <meta name="twitter:card" content="summary_large_image" />
      </Head>
      <article>{/* content */}</article>
    </>
  )
}
```

### Core Web Vitals Quick Checks

```
LCP  < 2.5s   Preload hero image, use fetchpriority="high"
INP  < 200ms  Break long tasks, yield to main thread
CLS  < 0.1    Set width/height on images, reserve space for dynamic content
```

### Image Optimization

```html
<!-- Modern formats with fallback -->
<picture>
  <source srcset="hero.avif" type="image/avif" />
  <source srcset="hero.webp" type="image/webp" />
  <img
    src="hero.jpg"
    alt="Descriptive alt text for the image"
    width="1200"
    height="630"
    fetchpriority="high"
  />
</picture>

<!-- Responsive images -->
<img
  srcset="photo-400.webp 400w, photo-800.webp 800w, photo-1200.webp 1200w"
  sizes="(max-width: 600px) 400px, (max-width: 900px) 800px, 1200px"
  src="photo-800.webp"
  alt="Descriptive alt text"
  width="1200"
  height="800"
  loading="lazy"
/>
```

## How to Use

Read individual rule files for detailed explanations and code examples:

```
rules/tech-meta-tags.md
rules/schema-json-ld.md
rules/cwv-lcp.md
rules/spa-rendering-strategy.md
rules/social-open-graph.md
```

Each rule file contains:
- YAML frontmatter with metadata (title, impact, tags)
- Brief explanation of why it matters
- Incorrect example with explanation
- Correct example with explanation
- Framework-specific examples (React, Laravel)

## References

- [Google Search Central](https://developers.google.com/search) - Official Google SEO documentation
- [web.dev Core Web Vitals](https://web.dev/articles/vitals) - CWV measurement and optimization
- [Schema.org](https://schema.org/) - Structured data vocabulary
- [Open Graph Protocol](https://ogp.me/) - Social sharing meta tags
- [Google Rich Results Test](https://search.google.com/test/rich-results) - Structured data validation

## Full Compiled Document

For the complete guide with all rules expanded: `AGENTS.md`
