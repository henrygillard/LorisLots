# Pitfalls Research

**Domain:** Shopify Theme Store — Dawn-based Liquid + Vanilla JS theme, migrating from Debut (legacy OS1.0)
**Researched:** 2026-05-06
**Confidence:** HIGH (Dawn architecture, Theme Store requirements, WCAG patterns are stable and well-documented; Debut migration patterns confirmed against actual codebase)

---

## Critical Pitfalls

### Pitfall 1: Carrying `{% include %}` Tags Into Dawn

**What goes wrong:**
Dawn uses `{% render %}` exclusively. `{% include %}` shares the parent scope (variables bleed between caller and snippet), whereas `{% render %}` is sandboxed. Copying Debut snippets into a Dawn build and leaving `{% include %}` intact means variables from the calling section leak into the snippet, producing subtle rendering bugs that only surface at runtime under certain data conditions.

**Why it happens:**
The Debut codebase uses `{% include %}` everywhere — 30+ call sites across sections and layout/theme.liquid (`{% include 'social-meta-tags' %}`, `{% include 'cart-popup' %}`, `{% include 'product-card-grid' %}`, `{% include 'site-nav' %}`, etc.). Developers migrating snippet content copy-paste and miss the tag change.

**How to avoid:**
1. At the start of each section build, do a `grep -r "include " sections/ snippets/ layout/` in the Dawn scaffold and treat any hit as a blocker before that file ships.
2. Use `{% render 'snippet-name', param: value %}` and pass all required variables explicitly as named parameters — never rely on ambient scope.
3. Configure a linting rule or pre-commit check for `{%- include` in any `.liquid` file.

**Warning signs:**
- Section renders correctly in isolation but shows wrong data when placed on a page alongside other sections.
- Snippet variables show unexpected values in Shopify Theme Editor preview.

**Phase to address:** Every section-build phase. Enforce at scaffold setup; check again at each new section.

---

### Pitfall 2: Client-Specific Content Baked Into Theme Liquid

**What goes wrong:**
This Debut theme contains collection-title string comparisons (`{% if collection.title == "Used DVDs" %}`), hardcoded marketing copy ("Lori's Lots carries over 10,000 used DVD titles..."), hardcoded `#333335` colors, and external image URLs (`src="https://i.imgur.com/HUCnCZQ.png"`) embedded directly in section templates. A Theme Store theme must work for any merchant's data. Reviewers will reject any theme that assumes specific collection names, product types, or merchant copy.

**Why it happens:**
The Debut build was built for one store, not a generic theme. These patterns worked perfectly for Loris Lots but are incompatible with Theme Store submission. Developers migrating piece by piece accidentally carry this logic forward.

**How to avoid:**
1. Every conditional that references a merchant-specific string (`collection.title ==`, `product.type ==`, `shop.name contains`) must be converted to a section setting, metafield, or removed.
2. Replace all hardcoded hex colors in `<style>` blocks with `settings.*` CSS custom properties (e.g., `var(--color-background)`).
3. External image URLs must become `section.settings.image` or `block.settings.image` pickers backed by Shopify CDN — never hotlinked from third-party hosts.
4. Marketing copy ("GET 12 DVDs for $25.97...") must become editable text settings in section schema, not hardcoded strings.

**Warning signs:**
- Any `{% if collection.title == "..." %}` or `{% if product.type == "..." %}` in a section.
- Any hex color in a `<style>` tag that does not reference `{{ settings.* }}`.
- Any `src="https://..."` image that is not `{{ section.settings.image | image_url }}`.
- Console.log statements left in production code (seen in `collection-template.liquid` line 413: `<script>console.log({{collection | json }})</script>`).

**Phase to address:** Collection page build, PDP build, and final Theme Store compliance pass. Run a grep for string literals in conditionals before each phase closes.

---

### Pitfall 3: Cart Drawer Race Conditions in Vanilla JS

**What goes wrong:**
A cart drawer that calls the Shopify AJAX Cart API (`/cart/add.js`, `/cart/update.js`, `/cart/change.js`) without request queuing can corrupt cart state. If a user adds two items rapidly, or if a network response is slow and the user clicks again, concurrent AJAX calls return stale cart data. The second `add.js` response may reflect a cart that doesn't include the first item yet, causing the drawer to render incorrect quantities.

**Why it happens:**
Dawn handles this via its `CartItems` and `CartDrawer` Web Components, which internally serialize requests using promise chains. When replacing these with Vanilla JS, developers write simple `fetch(...)` event listeners without tracking in-flight requests.

**How to avoid:**
1. Maintain a boolean `cartOperationInProgress` flag. Set it `true` when a request starts, reset in `.finally()`. Disable all cart-mutating buttons while `true`.
2. Better: implement a simple request queue — push operations to an array, run them serially with `async/await`.
3. After every mutating request, re-fetch `/cart.js` to get authoritative state before re-rendering. Never compute cart totals from partial responses.
4. Show a loading state (spinner or button disabled) during all cart operations.

**Warning signs:**
- Cart item count in header differs from drawer item list after rapid adds.
- Line item quantities jump unexpectedly on second click.
- Network tab shows two simultaneous POSTs to `/cart/add.js`.

**Phase to address:** Cart drawer implementation phase.

---

### Pitfall 4: Cart Drawer Accessibility Failures (Focus Trap and ARIA)

**What goes wrong:**
A cart drawer that opens but does not trap focus causes keyboard users to interact with content behind the drawer. Screen readers announce drawer content incorrectly if ARIA attributes are missing or wrong. Closing the drawer but failing to return focus to the trigger element fails WCAG 2.1 SC 2.4.3 (Focus Order).

**Why it happens:**
Dawn's `<cart-drawer>` Web Component implements focus trapping via `trapFocus` / `removeTrapFocus` helpers in `global.js`. When building in Vanilla JS without adopting those helpers, developers render the drawer visually but omit the keyboard/ARIA layer entirely.

**How to avoid:**
1. Copy Dawn's `trapFocus` / `removeTrapFocus` utility functions verbatim from Dawn's `global.js` — they are MIT-licensed and handle edge cases (first/last focusable element wrapping, Escape key).
2. The drawer element needs: `role="dialog"`, `aria-modal="true"`, `aria-label="Shopping cart"` (or `aria-labelledby` pointing to a heading inside the drawer).
3. The overlay/backdrop needs `aria-hidden="true"`.
4. On open: move focus to the first interactive element inside the drawer (close button).
5. On close: return focus to the element that triggered the open (typically the cart icon button). Store the trigger reference before opening.
6. Escape key must close the drawer from anywhere within it.
7. Add `inert` attribute to `<main>` and `<header>` when drawer is open (modern browsers) as belt-and-suspenders behind the focus trap.

**Warning signs:**
- Tab key cycles through page content while drawer is open.
- Screen reader does not announce drawer as a dialog.
- Closing drawer leaves focus at document body instead of cart trigger.

**Phase to address:** Cart drawer implementation phase. Re-verify in accessibility audit phase before Theme Store submission.

---

### Pitfall 5: JSON Template Migration Leaves `.liquid` Templates in Place

**What goes wrong:**
OS2.0 requires JSON templates (`templates/collection.json`, `templates/product.json`, etc.) so sections can be added via the Theme Editor. If the old `.liquid` templates from Debut (e.g., `templates/collection.liquid`, `templates/product.liquid`) are left alongside JSON templates, the `.liquid` file takes precedence in some Shopify contexts and the merchant cannot edit sections in the Theme Editor for those page types.

**Why it happens:**
Developers scaffold Dawn, then copy individual snippets/sections from Debut, but forget to remove the legacy `.liquid` template files they carried over from the export.

**How to avoid:**
1. Immediately after Dawn scaffold: verify the `templates/` directory contains ONLY JSON files (`collection.json`, `product.json`, `index.json`, `page.json`, `cart.json`, `404.json`, `password.json`, `search.json`).
2. Delete any `.liquid` files from `templates/` that have a JSON equivalent. (The `customers/` subdirectory still uses `.liquid` — keep those.)
3. Each JSON template's `"sections"` object must declare section IDs and the `"order"` array. Missing `"order"` means sections render in undefined sequence.

**Warning signs:**
- Theme Editor sidebar shows a page type but "Add section" does not appear.
- Sections added in Theme Editor do not persist after publish.
- `templates/` contains both `product.json` and `product.liquid`.

**Phase to address:** Dawn scaffold phase (first phase). Verify before any section work begins.

---

### Pitfall 6: `settings_schema.json` `theme_info` Block Fails Review

**What goes wrong:**
The Shopify Theme Store requires the first block in `settings_schema.json` to be a `theme_info` block with specific required fields: `theme_name`, `theme_author`, `theme_version`, `theme_documentation_url`, `theme_support_url`. The existing Debut `settings_schema.json` has these populated with Shopify's own values ("Debut", Shopify as author). Submitting with Shopify's name/URL as author will be rejected immediately.

**How to avoid:**
1. Update `theme_info` in `settings_schema.json` on Day 1 of scaffold:
   - `theme_name`: Your theme name (not "Debut")
   - `theme_author`: Your name or company
   - `theme_version`: Start at `"1.0.0"`
   - `theme_documentation_url`: A real URL you control (can be a GitHub Pages URL or placeholder that resolves)
   - `theme_support_url`: A real support URL

2. Bump `theme_version` on every significant change before resubmission. The review system tracks versions.

**Warning signs:**
- `settings_schema.json` still shows `"theme_author": "Shopify"`.
- `theme_documentation_url` points to `help.shopify.com`.

**Phase to address:** Dawn scaffold phase. Zero tolerance — fix before any other work.

---

### Pitfall 7: Section Schema Missing Required Fields or Using Unsupported Types

**What goes wrong:**
Sections that use unsupported `type` values in their `{% schema %}` block cause the Theme Editor to crash or display blank panels. Sections missing a `"name"` field at the top level silently fail schema validation. Blocks that lack `"type"` and `"name"` properties get rejected. The Theme Store reviewer tests every section in the Theme Editor.

**Why it happens:**
Developers building new sections copy-paste schema from Debut sections that predate certain OS2.0 schema requirements. Debut schemas were not written for the Theme Editor's full block UI — they predate the block limit system (`"max_blocks"`) and some input types.

**How to avoid:**
1. Every section schema must have: `"name"`, `"tag"` (optional but best practice), `"class"` (optional), `"settings": []`, and if blocks are used: `"blocks": [{ "type": "...", "name": "...", "settings": [] }]` plus `"max_blocks"`.
2. Valid input types (2025): `checkbox`, `number`, `radio`, `range`, `select`, `text`, `textarea`, `article`, `blog`, `collection`, `collection_list`, `color`, `color_background`, `color_scheme`, `color_scheme_group`, `font_picker`, `html`, `image_picker`, `inline_richtext`, `link_list`, `liquid`, `page`, `product`, `product_list`, `richtext`, `text_alignment`, `url`, `video`, `video_url`. Do not invent types.
3. `"presets"` block is required for sections to appear in the "Add section" panel in Theme Editor. Without it, a section can only be used if pre-listed in the JSON template.
4. Use `shopify theme check` (Shopify CLI) locally to validate schema before pushing.

**Warning signs:**
- Theme Editor sidebar shows section name but settings panel is empty.
- `shopify theme check` reports schema validation errors.
- Blocks cannot be added in Theme Editor despite `"blocks"` being defined.

**Phase to address:** Every section-build phase. Run `shopify theme check` as a phase gate.

---

### Pitfall 8: Debut's `data-section-type` JS Initialization Pattern vs Dawn's Web Components

**What goes wrong:**
Debut registers JavaScript behavior via `data-section-type` attributes and a global theme JS dispatcher that calls `theme.sections.register('section-type', SectionConstructor)`. Dawn uses Custom Elements (`customElements.define('cart-drawer', CartDrawer)`). If Vanilla JS for this project uses the Debut pattern (data attributes + global dispatcher), it won't integrate with Dawn's section lifecycle (`onBlockSelect`, `onBlockDeselect`, `onLoad`, `onUnload`) that the Theme Editor calls during live preview.

**Why it happens:**
Debut's `data-section-id` / `data-section-type` pattern is visible throughout the existing codebase (header, slideshow, product-template, etc.). Developers familiar with Debut replicate this pattern in the new theme.

**How to avoid:**
1. Use the Dawn section events API for Theme Editor integration: sections that need to respond to editor events (block select, section reorder) must implement `addEventListener` on `document` for `shopify:section:load`, `shopify:block:select`, etc.
2. For Vanilla JS without Web Components: wrap section logic in IIFE or module functions that are invoked on `DOMContentLoaded` AND re-invoked on `shopify:section:load` events. This mimics the lifecycle Dawn's Web Components handle automatically.
3. Do NOT carry forward the global `theme.sections` registry pattern — it relies on Debut's bundled `vendor.js` / `theme.js` which will not exist in the Dawn build.
4. The `data-section-id` and `data-section-type` attributes are not needed in Dawn and confuse maintenance.

**Warning signs:**
- Adding a block in Theme Editor does not re-render the section correctly in live preview.
- JS-powered carousels or tabs break after the merchant moves a section via drag-and-drop in the editor.

**Phase to address:** Dawn scaffold phase (establish the JS event pattern before any sections are built). Verify during Theme Editor QA at the end of each page section build.

---

### Pitfall 9: LCP Failures from Lazysizes / `data-src` Image Pattern

**What goes wrong:**
Debut's image rendering relies on `lazysizes.js` and `data-src` / `data-widths` attributes, which defer all image loading until JS parses and rewrites `src`. The Largest Contentful Paint image (hero or first product image) is never loaded eagerly, causing LCP scores of 4-8 seconds. Theme Store has a Core Web Vitals threshold; LCP above 2.5s is a rejection criterion.

**Why it happens:**
The existing `product-card-grid.liquid` snippet uses `class="grid-view-item__image lazyload"` + `data-src` for all images. This is the standard Debut pattern. Migrating the snippet directly copies this into Dawn.

**How to avoid:**
1. Use Dawn's `{% render 'image' %}` snippet (or equivalent) which outputs `<img>` with `srcset`, `sizes`, `width`, `height`, and `loading="lazy"` — but crucially uses `loading="eager"` + `fetchpriority="high"` for the first/hero image.
2. For the hero banner and first product card above the fold: never use `loading="lazy"`. Explicitly set `loading="eager" fetchpriority="high"`.
3. Use Shopify CDN image URLs with `image_url` filter and `width` parameter for proper `srcset` — not the old `img_url` filter (deprecated). `img_url` is still functional but the `image_url` + `image_tag` pattern is the current standard and produces better srcsets.
4. Eliminate `lazysizes.js` entirely. Modern `loading="lazy"` on `<img>` is supported in all Shopify-supported browsers.
5. Ensure hero and featured images are served at the correct display size — oversized images (2048px for a 600px container) inflate LCP transfer time.

**Warning signs:**
- Lighthouse LCP > 2.5s on collection or home page.
- Network tab shows hero image loaded after 800ms+ instead of starting immediately.
- Any `data-src` or `class="lazyload"` remaining in migrated image templates.

**Phase to address:** Every phase that renders images. Enforce at scaffold (remove lazysizes dependency). Verify with Lighthouse at end of each page phase.

---

### Pitfall 10: CLS from Unsized Images and Inline `<style>` in Sections

**What goes wrong:**
Cumulative Layout Shift occurs when images load without declared `width`/`height` (browser cannot reserve space) and when `<style>` blocks inside section `.liquid` files inject layout-changing CSS after initial paint. Both patterns appear extensively in the Debut codebase (the `collection-template.liquid` and `collection.liquid` both open with large `<style>` blocks injecting grid and card CSS inline).

**How to avoid:**
1. Every `<img>` must have explicit `width` and `height` attributes matching the image's native aspect ratio. Even with `width: 100%` CSS, the browser uses these to reserve aspect-ratio-correct space before the image loads.
2. Use Shopify's `image_tag` filter which auto-generates `width` and `height` from image metadata: `{{ image | image_url: width: 800 | image_tag }}`.
3. Move all section-specific CSS into the theme's main stylesheet (`assets/base.css` in Dawn) rather than inline `<style>` tags in section files. If section-specific CSS is needed, use `{{ 'section-name.css' | asset_url | stylesheet_tag }}` in the section file — but only load it when the section is present.
4. Never inject layout-critical CSS from JavaScript after DOMContentLoaded.

**Warning signs:**
- Lighthouse CLS > 0.1.
- Images pop in and push content down visibly on load.
- `<style>` blocks at the top of `.liquid` section files that set `width`, `height`, or `position` on layout elements.

**Phase to address:** Every phase. Enforce image sizing in product card snippet (built in collection phase). Verify CLS with Lighthouse before Theme Store submission.

---

### Pitfall 11: Blocking JavaScript Kills TTI

**What goes wrong:**
Debut loads `vendor.js` and `theme.js` with `defer` but also loads `lazysizes.js` with `async`. In a Dawn build, if Vanilla JS cart/filter/carousel code is bundled into a single large script and loaded without `defer` or `type="module"`, it blocks the main thread during parse, pushing Time to Interactive above the Theme Store threshold.

**How to avoid:**
1. All `<script>` tags in `layout/theme.liquid` must use `defer` or `type="module"` (which is implicitly deferred).
2. Do not load any JavaScript in the `<head>` without `defer` or `async` — even small scripts.
3. Split JS into per-feature files and load them only on pages where they are needed. Cart drawer JS should only load when the page contains a cart drawer element.
4. Do not inline large JS blocks in `layout/theme.liquid` — the existing `theme = { ... }` global object in the Debut layout is acceptable (it's small and sets up config), but anything larger than ~2KB of logic should be a separate deferred asset.

**Warning signs:**
- Lighthouse Total Blocking Time > 200ms.
- Performance timeline shows long tasks during initial script parse.
- Single JS file over 80KB minified.

**Phase to address:** Scaffold phase (establish loading pattern). Final performance audit.

---

### Pitfall 12: Accessibility on Product Title Link in Card Grid

**What goes wrong:**
The Debut `product-card-grid.liquid` pattern places an empty full-width anchor (`<a class="grid-view-item__link full-width-link">`) containing only `<span class="visually-hidden">{{ product.title }}</span>` as a "click anywhere on card" affordance, then separately renders `<div class="grid-view-item__title" aria-hidden="true">{{ product.title }}</div>`. This means screen readers encounter two representations of the product: one announced (the visually-hidden link text) and one suppressed (`aria-hidden`). The visible title is not inside the link. For keyboard users, the visible title is not a focusable element and the link target area is invisible.

**Why it happens:**
This was a common pattern in 2018-era Shopify themes for "whole-card is clickable" UX. Accessibility requirements have tightened.

**How to avoid:**
The correct OS2.0 pattern: wrap the product title in the anchor. The link should encompass the product image AND the product title text. Do not use a separate full-width invisible link plus an `aria-hidden` visible title. Do not use `aria-hidden` on meaningful product name text:
```html
<a href="{{ product.url }}">
  {{ product_image }}
  <span class="card-title">{{ product.title }}</span>
</a>
```
If a separate price or vendor element is outside the link, that is fine — only the image+title core should be wrapped.

**Warning signs:**
- `aria-hidden="true"` on a `<div>` containing the product title.
- A `<span class="visually-hidden">` inside an `<a>` that is otherwise empty.
- Screen reader announces product name then immediately announces the same name again.

**Phase to address:** Product card grid snippet (collection phase). This snippet is reused across home, collection, and search — fix once, applies everywhere.

---

### Pitfall 13: Skip-to-Content Link Rendered but Non-Functional

**What goes wrong:**
Dawn (and the Debut template) renders a "Skip to content" link (`<a href="#MainContent">`). Theme Store reviewers test this with keyboard. If `<main id="MainContent">` is missing, or if the element with that ID is not the first focusable/meaningful container after the header, the link either fails silently or focus lands in a wrong location. Debut's `theme.liquid` shows `id="MainContent"` on the `<main>` tag — this must be preserved exactly in the Dawn layout.

**How to avoid:**
1. `<main id="MainContent" role="main" tabindex="-1">` — the `tabindex="-1"` is required so programmatic focus (via the skip link's href) works even though `<main>` is not natively focusable.
2. The skip link must be the first element in `<body>`, visually hidden by default, visible on focus.
3. Do not rename or remove the `MainContent` ID during layout restructuring.

**Warning signs:**
- Tab once from address bar does not reveal a visible skip link.
- Clicking skip link does not move focus to main content area.
- `tabindex="-1"` missing from `<main>`.

**Phase to address:** Scaffold phase. Verify in every phase's accessibility checkpoint.

---

### Pitfall 14: Announcement Bar / Ticker Relying on Hardcoded Text and Infinite CSS Animation

**What goes wrong:**
The Debut `collection.liquid` and `collection-template.liquid` have `@keyframes animate` CSS with `translateX(0)` to `translateX(0)` — an animation with no net movement. The announcement text is hardcoded into HTML. Both problems must be fixed: the animation needs actual movement (or should be eliminated), and the text must be a Theme Editor setting. Additionally, continuously animating elements can reduce `prefers-reduced-motion` compliance and fail Theme Store accessibility checks.

**How to avoid:**
1. Convert announcement text to a section schema `text` or `richtext` setting so merchants can edit it.
2. Wrap any marquee/ticker animation in `@media (prefers-reduced-motion: no-preference)` so it only plays for users who have not requested reduced motion. For users with reduced motion preference, show the text statically.
3. If implementing a scrolling ticker: use CSS `animation` with actual `translateX(-50%)` movement on a doubled text content. Test that the text loops seamlessly.

**Warning signs:**
- Hardcoded announcement text in section `.liquid` files.
- `@keyframes` with identical start and end `transform` values.
- No `prefers-reduced-motion` media query around animation.

**Phase to address:** Home page and collection page section builds.

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Inline `<style>` in section `.liquid` | Easy scoped styling without separate CSS file | Styles load on every page; not deduplicated; CLS risk; harder to maintain | Never for layout-impacting styles; acceptable only for truly section-unique, non-layout CSS on sections that are rarely used |
| Single bundled `theme.js` | Simple deployment | Loads code for features not on current page; larger TTI | Acceptable for MVP if total size < 50KB minified; refactor if TBT > 200ms |
| `{% render %}` with no parameter documentation | Faster to write | Next developer cannot understand what variables are expected | Never for shared snippets (product card, price, media); document all render parameters |
| Hardcoded breakpoint values in JS | Simpler JS | Out of sync with CSS breakpoints; bugs on edge screens | Never — use CSS custom properties or a single shared breakpoint config |
| `console.log` left in production liquid | Useful in dev | Leaks store data structure to browser console; reviewers will catch it | Never in any file committed to the theme repo |
| External CDN images (imgur, etc.) | Quick during dev | Broken images if CDN changes; fails Theme Store review; merchant cannot change them | Never in theme templates — use Shopify CDN via `image_url` filter |

---

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| Shopify AJAX Cart API | Reading cart state from the `add.js` response body instead of re-fetching `/cart.js` | Always re-fetch `/cart.js` after any mutation to get authoritative cart state |
| Shopify AJAX Cart API | Not handling 422 errors (variant unavailable, sold out) — fetch throws but UI shows success | Wrap all cart fetch calls in try/catch; check `response.ok`; show error message to user |
| Shopify Storefront section rendering | Calling `sections.render` on a section that has no `"presets"` in its schema | Add a preset to every section that should be dynamically renderable; test with `fetch('/variants/X.json?section_id=Y')` pattern |
| Shopify predictive search API | Calling `/search/suggest.json` without a `resources[type]` parameter returns all resource types, causing unexpectedly large responses | Always pass `resources[type]=product` (and `article`, `page` if needed) to limit response |
| Theme Editor live preview | Vanilla JS that runs once on `DOMContentLoaded` without re-running on `shopify:section:load` | Always add `document.addEventListener('shopify:section:load', handler)` that re-initializes JS-powered components |
| Variant URL routing | Using `window.location.hash` for variant selection state (Debut pattern) | Dawn uses `?variant=` query param; use `history.pushState` with query params, not hash |

---

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| `img_url` without explicit width | Images served at full resolution regardless of display size | Use `image_url: width: X` to request appropriately sized image; never use bare `img_url` or `img_url: '2048x'` for thumbnails | From the very first page load |
| Unthrottled scroll or resize event listeners | Janky scroll, high CPU usage on mobile | Debounce all scroll/resize handlers; use `IntersectionObserver` instead of scroll for lazy behavior | Immediately on low-power devices |
| `document.querySelectorAll` in a tight loop | Perceptible delay on collection pages with 48+ products | Cache DOM queries; use event delegation on a container instead of listeners on each card | ~24+ product cards per page |
| Loading section-specific JS on every page via `theme.liquid` | Increased TTI on pages that don't use that feature | Load JS inside the section `.liquid` file that uses it, not globally in `layout/theme.liquid` | Always; cost scales with number of features |
| Re-rendering entire cart drawer HTML on every update | Flicker, CLS, lost focus state during cart updates | Re-render only changed line items; or use a stable DOM structure and update text/count nodes in place | Visible on every cart interaction |

---

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| Outputting `{{ product.description }}` without `html_safe` consideration | XSS if a compromised admin account injects script via product description | Shopify's Liquid engine escapes `{{ }}` by default; `{{ variable }}` is safe. Risk only exists with `{{ variable | raw }}` — never use `raw` filter on merchant content |
| Exposing full product JSON in `<script>` via `{{ product | json }}` | Leaks internal pricing, metafields, variant data not displayed in UI | Only serialize the variant data needed for JS (`product.variants | json` scoped to `id`, `price`, `available`, `option_values`) |
| Third-party scripts loaded from external CDN without SRI | Supply chain attack via compromised CDN | Avoid third-party scripts entirely (Theme Store requirement: no unapproved external scripts). If required, use Subresource Integrity |
| Cart token accessible in client JS | Low risk in isolation; combined with CSRF enables cart manipulation | Cart tokens are intentionally client-accessible in Shopify's model; no action needed, but don't store them in `localStorage` |

---

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Cart drawer opens but no scroll lock on body | Page scrolls behind drawer; disorienting on mobile | Set `document.body.style.overflow = 'hidden'` when drawer opens; restore on close |
| Sticky ATC bar on PDP overlaps footer or fixed bottom nav on mobile | Obscures content; can hide price information | Test sticky ATC at 375px viewport; ensure it does not overlap the footer when user reaches bottom of page; use `position: sticky` with `bottom: 0` and test with iOS Safari's variable toolbar height |
| Pill filter bar on collection page does not show active filter state | Users do not know which filter is applied; leads to confusion and back-navigation | Active pill must have clear visual differentiation (filled background, ARIA `aria-pressed="true"`) and a clear/remove affordance |
| Format tab navigation on home page only works with mouse | Keyboard-only and mobile users cannot switch tabs | Use `role="tablist"` / `role="tab"` pattern; handle ArrowLeft/ArrowRight keyboard navigation within the tab list |
| "Add to cart" button on product card (collection grid) without variant selector | Silently adds wrong variant if product has variants | Either suppress the "quick add" button for multi-variant products, or implement a variant picker modal before adding |

---

## "Looks Done But Isn't" Checklist

- [ ] **Cart drawer:** Verify focus is trapped inside drawer with Tab key — verify Escape key closes it — verify focus returns to cart trigger button after close.
- [ ] **Product card grid:** Verify `<img>` elements have `width` and `height` attributes — verify no `data-src` or `class="lazyload"` attributes remain.
- [ ] **Collection pill filters:** Verify active filter has `aria-pressed="true"` — verify filter works without JavaScript (graceful degradation via URL params).
- [ ] **Announcement/ticker:** Verify text is editable in Theme Editor — verify animation respects `prefers-reduced-motion`.
- [ ] **JSON templates:** Verify `templates/` directory contains no `.liquid` files that have a `.json` counterpart.
- [ ] **Schema:** Run `shopify theme check` — zero errors, zero warnings related to schema.
- [ ] **Theme Editor:** Add, move, and remove every section in Theme Editor without JS errors — verify live preview updates correctly.
- [ ] **settings_schema.json:** Verify `theme_author` is not "Shopify" — verify `theme_documentation_url` resolves.
- [ ] **No client-specific content:** Grep for `collection.title ==`, hardcoded hex colors in `<style>` blocks, `src="https://i.imgur.com"`, `console.log`.
- [ ] **Skip link:** Press Tab once on every page type — verify skip link appears — verify Enter activates it and moves focus to `<main>`.
- [ ] **Reduced motion:** Disable animation in OS settings — verify no animated elements still animate.
- [ ] **Mobile viewport:** Test all pages at 375px width on iOS Safari — verify no horizontal overflow.

---

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| `{% include %}` used throughout instead of `{% render %}` | MEDIUM | Systematic grep-and-replace; verify each call site passes required variables explicitly; test section by section |
| Client-specific content found during Theme Store review | HIGH | Identify all instances with grep; convert each to section schema settings; requires re-testing Theme Editor for each affected section |
| Cart drawer race conditions discovered in QA | MEDIUM | Implement request queue (2-3 day effort); requires full cart interaction regression test |
| JSON template missing causing sections not editable | LOW | Add/replace `.liquid` template with correct `.json` template; update `sections` and `order` fields; test in Theme Editor |
| LCP failure discovered during performance audit | MEDIUM | Audit every image render call in templates; add `loading="eager" fetchpriority="high"` to above-fold images; remove lazysizes; requires Lighthouse re-run |
| `settings_schema.json` rejected by Theme Store | LOW | Update `theme_info` fields; bump version; resubmit — turnaround 1-3 days |
| Focus trap missing from cart drawer | MEDIUM | Copy `trapFocus` / `removeTrapFocus` utilities from Dawn `global.js`; wire to drawer open/close events; requires keyboard QA |

---

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| `{% include %}` vs `{% render %}` | Phase 1: Dawn Scaffold | Grep for `{% include %}` in any `.liquid` file; zero hits required |
| Client-specific content in templates | Phases 2-5 (each page build) + Final | Grep for `collection.title ==`, `collection.handle ==`, hardcoded hex, external image URLs |
| Cart drawer race conditions | Phase 5: Cart Drawer | Manual rapid-add test; network tab inspection |
| Cart drawer focus trap | Phase 5: Cart Drawer | Keyboard-only walkthrough; screen reader test |
| JSON template / `.liquid` template conflict | Phase 1: Dawn Scaffold | `ls templates/` — confirm only `.json` files (except `customers/`) |
| `settings_schema.json` theme_info | Phase 1: Dawn Scaffold | Visual inspection of first block; verify URLs resolve |
| Section schema missing fields | Phases 2-5 (each page build) | `shopify theme check` after each section; Theme Editor panel opens correctly |
| Debut `data-section-type` JS pattern | Phase 1: Dawn Scaffold | Grep for `data-section-type` in any new file; use Dawn section events instead |
| LCP from lazy-loaded hero images | Phase 1 (remove lazysizes); Phases 2-4 (image markup) | Lighthouse LCP < 2.5s on collection and home page |
| CLS from unsized images | Phases 2-5 (each image) | Lighthouse CLS < 0.1; all `<img>` have `width` and `height` |
| Blocking JS | Phase 1: Scaffold JS loading | No `<script>` in `<head>` without `defer`; TBT < 200ms in Lighthouse |
| Product title link accessibility | Phase 3: Collection page card | Axe DevTools on collection page; no `aria-hidden` on visible product title |
| Skip-to-content link | Phase 1: Scaffold | Tab key test on every page type |
| Announcement ticker | Phases 2-3: Home + Collection | `prefers-reduced-motion` OS setting test; Theme Editor text field visible |

---

## Sources

- Shopify Theme Store requirements reviewed against official Shopify documentation (shopify.dev/docs/storefronts/themes/store/requirements) — HIGH confidence, stable requirements
- Dawn `global.js` focus trap utilities reviewed in Dawn v14.x (GitHub: Shopify/dawn) — HIGH confidence
- WCAG 2.1 SC 2.1.2 (No Keyboard Trap), 2.4.3 (Focus Order), 1.4.3 (Contrast), 4.1.2 (Name, Role, Value) — HIGH confidence
- Debut codebase (this repository) — direct inspection of `{% include %}` usage, `data-section-type` pattern, `lazysizes.js`, `data-src` image pattern, hardcoded colors, client-specific conditionals, console.log — HIGH confidence (observed directly)
- Shopify AJAX Cart API behavior (race conditions, 422 responses) — HIGH confidence, documented behavior
- Core Web Vitals thresholds (LCP < 2.5s good, > 4s poor; CLS < 0.1 good; TBT < 200ms good) — HIGH confidence, Google/Shopify documented
- `prefers-reduced-motion` media query — HIGH confidence, W3C specification

---
*Pitfalls research for: Dawn-based Shopify Theme Store theme, migrating from Debut*
*Researched: 2026-05-06*
