---
title: Provide Meaningful Alt Text for Images
impact: CRITICAL
impactDescription: WCAG 2.1 Level A - Required for non-text content
tags: accessibility, images, alt-text, screen-readers
---

## Provide Meaningful Alt Text for Images

**Impact: CRITICAL (WCAG 2.1 Level A - Required for non-text content)**

Provide meaningful alternative text for images that conveys the purpose and content of the image to users who cannot see it.

## Bad Example

```html
<!-- Anti-pattern: Missing or poor alt text -->
<img src="hero.jpg">

<img src="logo.png" alt="logo">

<img src="team-photo.jpg" alt="image">

<img src="chart.png" alt="chart.png">

<img src="product.jpg" alt="Click here to buy now!">

<!-- Decorative image with alt text -->
<img src="decorative-border.png" alt="decorative border image">

<!-- Complex image with inadequate description -->
<img src="infographic.png" alt="infographic">

<!-- Icon with redundant alt -->
<a href="/search">
  <img src="search-icon.svg" alt="search icon">
  Search
</a>
```

## Good Example

```html
<!-- Correct approach: Meaningful, contextual alt text -->

<!-- Informative image: describe content and purpose -->
<img src="hero.jpg" alt="Team of developers collaborating around a whiteboard filled with code diagrams">

<!-- Logo: include company name -->
<img src="logo.png" alt="Acme Corporation">

<!-- Team photo: describe relevant details -->
<img src="team-photo.jpg" alt="Our 12-person development team at the 2024 company retreat in Colorado">

<!-- Chart: describe the data and insights -->
<img src="chart.png" alt="Line chart showing website traffic growth from 10,000 monthly visitors in January to 45,000 in December 2024">

<!-- Product image: describe the product -->
<img src="product.jpg" alt="Blue wireless headphones with cushioned ear cups and adjustable headband">

<!-- Decorative image: use empty alt -->
<img src="decorative-border.png" alt="" role="presentation">

<!-- Complex image: use figure and detailed description -->
<figure>
  <img src="infographic.png" alt="2024 Industry trends infographic (detailed description below)">
  <figcaption>
    <details>
      <summary>Detailed description of infographic</summary>
      <p>This infographic shows five key industry trends for 2024:</p>
      <ol>
        <li>AI adoption increased 45% year over year</li>
        <li>Remote work stabilized at 35% of workforce</li>
        <li>Cybersecurity spending grew by 20%</li>
        <li>Cloud migration reached 80% completion</li>
        <li>Sustainability initiatives in 60% of companies</li>
      </ol>
    </details>
  </figcaption>
</figure>

<!-- Icon with text: hide redundant icon -->
<a href="/search">
  <img src="search-icon.svg" alt="" aria-hidden="true">
  Search
</a>

<!-- Functional image: describe the action -->
<button>
  <img src="print-icon.svg" alt="Print this page">
</button>

<!-- Image as link: describe destination -->
<a href="/products/headphones">
  <img src="headphones-thumb.jpg" alt="View Blue Wireless Headphones product details">
</a>

<!-- Background image with important content -->
<div class="hero" style="background-image: url('hero-bg.jpg');" role="img" aria-label="Aerial view of San Francisco skyline at sunset">
  <h1>Welcome to San Francisco</h1>
</div>
```

## Why

Alt text is critical for accessibility and SEO:

1. **Screen Reader Users**: Alt text is read aloud by screen readers, providing context for visual content.

2. **Image Load Failures**: Alt text displays when images fail to load.

3. **Search Engines**: Alt text helps search engines understand image content.

4. **Cognitive Disabilities**: Alt text can help users understand the purpose of images.

5. **Legal Compliance**: WCAG requires text alternatives for non-text content.

Alt text guidelines:

**Informative Images:**
- Describe the content and function
- Be concise but complete (usually under 125 characters)
- Don't start with "Image of" or "Picture of"
- Include relevant details based on context

**Decorative Images:**
- Use empty alt (`alt=""`)
- Add `role="presentation"` for clarity
- CSS background images are inherently decorative

**Functional Images (buttons/links):**
- Describe the action, not the appearance
- "Submit form" not "Green button"

**Complex Images (charts, diagrams):**
- Provide brief alt text
- Offer detailed description via `<figcaption>`, link, or `aria-describedby`

**Images of Text:**
- Include all the text in the alt
- Better: use actual text instead of images

**Image Links:**
- Describe the destination
- "View product details" not "thumbnail image"

Testing tip: Ask yourself: "If I couldn't see this image, what information would I need?"
