# Mobile-First Responsive Design

Design for mobile devices first, then progressively enhance for larger screens using Tailwind's responsive prefixes.

## Bad Example

```html
<!-- Desktop-first approach (anti-pattern) -->
<div class="flex flex-row lg:flex-col md:flex-col sm:flex-col">
  <div class="w-1/4 lg:w-full md:w-full sm:w-full">Sidebar</div>
  <div class="w-3/4 lg:w-full md:w-full sm:w-full">Content</div>
</div>

<!-- Overriding desktop styles for mobile -->
<p class="text-xl sm:text-base">
  Text that needs to be smaller on mobile
</p>
```

## Good Example

```html
<!-- Mobile-first approach -->
<div class="flex flex-col lg:flex-row">
  <div class="w-full lg:w-1/4">Sidebar</div>
  <div class="w-full lg:w-3/4">Content</div>
</div>

<!-- Start small, scale up -->
<p class="text-base lg:text-xl">
  Text that grows on larger screens
</p>

<!-- Progressive enhancement for navigation -->
<nav class="flex flex-col space-y-2 md:flex-row md:space-y-0 md:space-x-4">
  <a href="#">Home</a>
  <a href="#">About</a>
  <a href="#">Contact</a>
</nav>
```

## Why

1. **Tailwind's breakpoints are min-width based**: Unprefixed utilities apply to all screen sizes, while `sm:`, `md:`, `lg:`, `xl:`, and `2xl:` apply at that breakpoint and above.

2. **Smaller CSS bundle**: Mobile-first requires fewer overrides since you're enhancing rather than undoing styles.

3. **Better performance on mobile**: Mobile devices receive simpler styles without processing unnecessary desktop overrides.

4. **Follows natural content flow**: Content naturally stacks vertically, which is ideal for mobile layouts.

5. **Easier maintenance**: Progressive enhancement is more intuitive than progressive degradation.

## Breakpoint Reference

| Prefix | Min-width | CSS |
|--------|-----------|-----|
| `sm:`  | 640px     | `@media (min-width: 640px)` |
| `md:`  | 768px     | `@media (min-width: 768px)` |
| `lg:`  | 1024px    | `@media (min-width: 1024px)` |
| `xl:`  | 1280px    | `@media (min-width: 1280px)` |
| `2xl:` | 1536px    | `@media (min-width: 1536px)` |
