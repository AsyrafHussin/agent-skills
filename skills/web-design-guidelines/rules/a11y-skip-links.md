---
title: Provide Skip Links for Navigation
impact: HIGH
impactDescription: WCAG 2.1 Level A - Bypass blocks
tags: accessibility, navigation, keyboard, skip-links
---

## Provide Skip Links for Navigation

**Impact: HIGH (WCAG 2.1 Level A - Bypass blocks)**

Provide skip links to allow keyboard and screen reader users to bypass repetitive content and navigate directly to main content areas.

## Bad Example

```html
<!-- Anti-pattern: No skip links -->
<!DOCTYPE html>
<html>
<head>
  <title>My Website</title>
</head>
<body>
  <header>
    <nav>
      <a href="/">Home</a>
      <a href="/about">About</a>
      <a href="/services">Services</a>
      <a href="/products">Products</a>
      <a href="/portfolio">Portfolio</a>
      <a href="/blog">Blog</a>
      <a href="/contact">Contact</a>
      <!-- 20+ more links in mega menu -->
    </nav>
  </header>
  <main>
    <h1>Welcome</h1>
    <!-- User must tab through ALL nav links to reach this content -->
  </main>
</body>
</html>
```

## Good Example

```html
<!-- Correct approach: Comprehensive skip links -->
<!DOCTYPE html>
<html>
<head>
  <title>My Website</title>
  <style>
    .skip-links {
      position: absolute;
      top: 0;
      left: 0;
      z-index: 9999;
    }

    .skip-link {
      position: absolute;
      left: -10000px;
      top: auto;
      width: 1px;
      height: 1px;
      overflow: hidden;
      background: #000;
      color: #fff;
      padding: 1rem;
      text-decoration: none;
      font-weight: bold;
    }

    .skip-link:focus {
      position: fixed;
      top: 0;
      left: 0;
      width: auto;
      height: auto;
      overflow: visible;
      outline: 3px solid #005fcc;
      outline-offset: 2px;
    }

    /* Smooth scroll for skip link targets */
    html {
      scroll-behavior: smooth;
    }

    /* Ensure skip link targets are focusable */
    [id]:target {
      scroll-margin-top: 1rem;
    }
  </style>
</head>
<body>
  <div class="skip-links">
    <a href="#main-content" class="skip-link">Skip to main content</a>
    <a href="#main-nav" class="skip-link">Skip to navigation</a>
    <a href="#search" class="skip-link">Skip to search</a>
    <a href="#footer" class="skip-link">Skip to footer</a>
  </div>

  <header>
    <form id="search" role="search" tabindex="-1">
      <label for="search-input" class="visually-hidden">Search</label>
      <input type="search" id="search-input" placeholder="Search...">
      <button type="submit">Search</button>
    </form>

    <nav id="main-nav" aria-label="Main navigation" tabindex="-1">
      <a href="/">Home</a>
      <a href="/about">About</a>
      <a href="/services">Services</a>
      <a href="/products">Products</a>
      <a href="/portfolio">Portfolio</a>
      <a href="/blog">Blog</a>
      <a href="/contact">Contact</a>
    </nav>
  </header>

  <main id="main-content" tabindex="-1">
    <h1>Welcome to Our Website</h1>
    <p>Main content starts here...</p>

    <!-- For long pages, provide in-page skip links -->
    <nav aria-label="Page sections">
      <h2>On this page</h2>
      <ul>
        <li><a href="#section-1">Introduction</a></li>
        <li><a href="#section-2">Features</a></li>
        <li><a href="#section-3">Pricing</a></li>
      </ul>
    </nav>

    <section id="section-1" tabindex="-1">
      <h2>Introduction</h2>
      <!-- Content -->
    </section>

    <section id="section-2" tabindex="-1">
      <h2>Features</h2>
      <!-- Content -->
    </section>

    <section id="section-3" tabindex="-1">
      <h2>Pricing</h2>
      <!-- Content -->
    </section>
  </main>

  <footer id="footer" tabindex="-1">
    <p>&copy; 2024 My Website</p>
  </footer>
</body>
</html>
```

## Why

Skip links are a WCAG requirement and essential for usability:

1. **Keyboard Efficiency**: Without skip links, keyboard users must tab through every navigation link on every page load to reach main content.

2. **Screen Reader Navigation**: While screen reader users can navigate by headings, skip links provide a direct, consistent mechanism.

3. **Repetitive Content**: Headers, navigation, sidebars appear on every page. Users shouldn't have to traverse them repeatedly.

4. **Motor Disabilities**: Reducing the number of key presses helps users with motor impairments.

5. **WCAG Requirement**: WCAG 2.1 Success Criterion 2.4.1 requires a mechanism to bypass repeated content.

Skip link best practices:

1. **First Focusable Element**: Skip link should be the very first focusable element
2. **Visible on Focus**: Hidden until focused, then prominently displayed
3. **Multiple Skip Links**: Consider links to main content, navigation, search, footer
4. **Target Focusable**: Add `tabindex="-1"` to skip link targets for browser compatibility
5. **Clear Labels**: "Skip to main content" is clearer than "Skip navigation"
6. **Consistent Placement**: Same location on every page
7. **High Contrast**: Ensure skip link is visible when focused
8. **Table of Contents**: For long pages, provide in-page navigation

Note: Some browsers don't move focus correctly to skip link targets without `tabindex="-1"` on the target element.
