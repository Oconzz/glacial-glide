# Assessment Changes

This document outlines the changes made to complete the three assessment tasks.

Three tasks were completed on the `assessment` branch: a variant update bug fix, a product image carousel implementation, and a related products section. Below is a summary of what was done and why.

## Task 1 – Related Products Section

The `related-products` section already existed in the codebase but wasn't being rendered on any product page. I activated it by adding it to `templates/product.json` and also added `presets` to its schema. Without presets, the section doesn't appear as an option in the Shopify theme editor customizer, so merchants can't add or remove it via drag-and-drop.

Products are auto-populated through Shopify's native recommendations API, so no manual curation is needed. The heading text, number of products, columns, image style, and spacing are all configurable from the theme editor without touching code.

I also improved the section's CSS slightly. The heading was left-aligned with no responsive treatment, so I centred it and added proper grid spacing using the theme's existing CSS variables to keep things consistent.

Since the recommendations API needs some data to work properly (purchase history, tags, etc.), I added a fallback that shows products from the same collection when there are no recommendations available. This is especially useful for new stores or products that don't have enough data yet.

**Files changed:**

- `templates/product.json` — added section to the page and the order array
- `sections/related-products.liquid` — added `presets` to the schema + fallback for same-collection products
- `assets/section-related-products.css` — centred heading, responsive grid gap

## Task 2 – Product Image Carousel with Thumbnails

The original `product-media-gallery.liquid` was a plain `<div>` loop that just rendered images one after another with no interactive structure. The JavaScript file `media-gallery.js`, which was already included in the page, expects a specific custom element structure with `<media-gallery>`, `<slider-component>`, and `data-media-id` attributes on each slide to wire up the carousel behaviour, thumbnail clicks, and variant image sync. None of that was present, so the JS was effectively doing nothing.

I rebuilt the snippet to match the structure the JS expects:

- A `<media-gallery>` wrapper that `media-gallery.js` binds to on load
- A `<slider-component id="GalleryViewer-...">` for the main image viewer with each media item as a `<li data-media-id="...">` slide
- A `<slider-component id="GalleryThumbnails-...">` for the thumbnail strip, with each thumbnail as a `<li data-target="...">` containing a clickable `<button>`
- Prev/Next navigation buttons, a slide counter, and an accessibility live region

The featured media (variant's image) is rendered first and gets the `is-active` class. Swipe on mobile works via the native CSS `scroll-snap` already built into `slider-component`. When a variant is selected, `product-info.js` calls `setActiveMedia()` on the `<media-gallery>` element, which scrolls the viewer and updates the active thumbnail. No extra code needed on my end since that wiring was already there.

I also had to fix a few things during testing. The Liquid `image_tag` filter doesn't allow filter chains inside named parameters, so I had to pre-assign the thumbnail IDs with `assign` before passing them to `image_tag`. I also added CSS rules with `!important` to ensure only the active slide is visible at any time, and added a fallback so the first image always shows even if the variant doesn't have a featured media assigned.

**Files changed:**

- `snippets/product-media-gallery.liquid` — full rebuild
- `assets/section-main-product.css` — added `!important` to `.is-active` display rules

## Task 3 – Variant Detail Update Bug Fix

When changing variants, the page was always showing "Unavailable" and hiding price, SKU, and inventory, regardless of actual stock. Tracing the issue back, `getSelectedVariant()` in `product-info.js` was returning `null` every time, which caused `setUnavailable()` to be called on every variant change.

The problem was a one-character CSS selector mistake:

```javascript
// before, tries to match variant-selects element itself having the attribute
querySelector("variant-selects[data-selected-variant]");

// after, correctly targets the child script element inside variant-selects
querySelector("variant-selects [data-selected-variant]");
```

In `product-variant-picker.liquid`, `data-selected-variant` lives on a `<script type="application/json">` inside `<variant-selects>`, not on `<variant-selects>` itself. The missing space meant the selector never matched.

**Files changed:**

- `assets/product-info.js` — line 141, one space added
