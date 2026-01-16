---
title: Maintain Proper Heading Hierarchy
impact: CRITICAL
impactDescription: WCAG 2.1 Level A - Info and Relationships
tags: accessibility, headings, structure, navigation
---

## Maintain Proper Heading Hierarchy

**Impact: CRITICAL (WCAG 2.1 Level A - Info and Relationships)**

Use heading elements (h1-h6) in a logical, hierarchical order to create a clear document outline. Proper heading structure is essential for screen reader navigation and SEO.

## Bad Example

```html
<!-- Anti-pattern: Broken heading hierarchy -->
<body>
  <!-- Skipping h1 -->
  <h2>Welcome to Our Store</h2>

  <div class="hero">
    <!-- Skipping levels (h2 to h4) -->
    <h4>Shop the Latest Collection</h4>
  </div>

  <section>
    <!-- Using h1 for styling purposes -->
    <h1 class="small-heading">Featured Products</h1>

    <div class="product">
      <!-- Heading levels chosen for styling, not structure -->
      <h5>Product Name</h5>
      <h6>$29.99</h6>
    </div>
  </section>

  <aside>
    <!-- Multiple h1s on the page -->
    <h1>Special Offers</h1>
  </aside>

  <!-- Using heading for non-heading content -->
  <h3>© 2024 Our Company</h3>
</body>
```

## Good Example

```html
<!-- Correct approach: Logical heading hierarchy -->
<body>
  <header>
    <a href="/" aria-label="Home">
      <img src="logo.svg" alt="Acme Store">
    </a>
  </header>

  <!-- Single h1 for main page title -->
  <main>
    <h1>Welcome to Acme Store</h1>

    <section aria-labelledby="hero-heading">
      <h2 id="hero-heading">Shop the Latest Collection</h2>
      <p>Discover our new arrivals for the season.</p>
    </section>

    <section aria-labelledby="featured-heading">
      <h2 id="featured-heading">Featured Products</h2>

      <article class="product">
        <h3>Wireless Headphones</h3>
        <p class="price">$29.99</p>
        <p>High-quality audio with 20-hour battery life.</p>
      </article>

      <article class="product">
        <h3>Smart Watch</h3>
        <p class="price">$199.99</p>
        <p>Track your fitness and stay connected.</p>

        <!-- Sub-sections within product -->
        <section>
          <h4>Key Features</h4>
          <ul>
            <li>Heart rate monitor</li>
            <li>GPS tracking</li>
            <li>Water resistant</li>
          </ul>
        </section>

        <section>
          <h4>Customer Reviews</h4>
          <div class="review">
            <h5>Great product!</h5>
            <p>Exceeded my expectations...</p>
          </div>
        </section>
      </article>
    </section>

    <section aria-labelledby="categories-heading">
      <h2 id="categories-heading">Shop by Category</h2>

      <div class="category">
        <h3>Electronics</h3>
        <ul>
          <li><a href="/electronics/phones">Phones</a></li>
          <li><a href="/electronics/tablets">Tablets</a></li>
        </ul>
      </div>

      <div class="category">
        <h3>Clothing</h3>
        <ul>
          <li><a href="/clothing/mens">Men's</a></li>
          <li><a href="/clothing/womens">Women's</a></li>
        </ul>
      </div>
    </section>
  </main>

  <aside aria-labelledby="offers-heading">
    <h2 id="offers-heading">Special Offers</h2>
    <p>Get 20% off your first order!</p>
  </aside>

  <footer>
    <!-- Non-heading content styled differently -->
    <p><small>&copy; 2024 Acme Store. All rights reserved.</small></p>
  </footer>
</body>

<style>
/* Style headings independently of their level */
.product h3 {
  font-size: 1.25rem;
  font-weight: 600;
}

.product h4 {
  font-size: 1rem;
  font-weight: 500;
  color: #666;
}

/* Utility for visually styling any element as a heading */
.looks-like-h2 {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 1rem;
}
</style>
```

## Why

Proper heading hierarchy is fundamental for accessibility:

1. **Screen Reader Navigation**: Users navigate by headings (H key). They rely on the hierarchy to understand document structure and jump to sections.

2. **Document Outline**: Headings create an outline that helps all users understand content organization.

3. **SEO**: Search engines use heading hierarchy to understand page structure and importance.

4. **Cognitive Accessibility**: Clear structure helps users with cognitive disabilities process information.

5. **WCAG Compliance**: Success Criterion 1.3.1 requires proper heading structure.

Heading hierarchy rules:

1. **One h1 per Page**: Use h1 for the main page title or topic
2. **Don't Skip Levels**: Go h1 → h2 → h3 (not h1 → h3)
3. **Nest Properly**: h3 should be inside h2 section, etc.
4. **Semantic, Not Visual**: Choose heading level based on structure, not appearance
5. **Style with CSS**: Use CSS to control visual appearance of any heading level
6. **Headings for Structure**: Don't use headings just for bold text

Screen reader heading navigation:
- **H key**: Next heading
- **Shift+H**: Previous heading
- **1-6 keys**: Jump to specific heading level
- **Insert+F6** (JAWS): Heading list

Testing your heading structure:
- Use browser extensions like HeadingsMap
- Use screen reader heading navigation
- Review the document outline
- Check WAVE accessibility tool

Remember: The visual size and style of headings should be controlled by CSS, not by choosing inappropriate heading levels.
