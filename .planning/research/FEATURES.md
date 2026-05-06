# Feature Research

**Domain:** Shopify OS2.0 Theme — Home, Collection, PDP, Cart pages
**Researched:** 2026-05-06
**Confidence:** MEDIUM (Theme Store review requirements from training data, stable since OS2.0 rollout 2021–2022; wireframe analysis HIGH confidence from direct file inspection)

---

## Confidence Notes

Theme Store review criteria are based on training data current through August 2025. Shopify has not publicly changed the core requirements since the OS2.0 transition, but the exact current checklist should be verified at https://shopify.dev/docs/storefronts/themes/store/requirements before final submission. All wireframe-derived findings are HIGH confidence (files directly inspected).

---

## Page 1: Home Page (Home B — Editorial Hero)

### Table Stakes (Theme Store will flag if missing)

| Feature | Why Expected / Required | Complexity | Notes |
|---------|------------------------|------------|-------|
| Sections Everywhere (OS2.0 JSON template) | Theme Store hard requirement — `templates/index.json` must use JSON with sections, not `index.liquid` | LOW | Dawn already does this; don't regress it |
| Theme editor customization for every section | Reviewers test that every visible element has a schema setting; hard-coded content fails review | MEDIUM | Every section needs `{% schema %}` with color, text, image, and toggle settings |
| Announcement bar / promo ticker | Merchants universally expect a top-of-page promotional message strip | LOW | Wireframe shows a scrolling ticker; a static bar also satisfies this |
| Navigation header with logo, nav links, cart icon | Non-negotiable for any storefront — reviewers test all nav states | LOW | Dawn's header section is the base; customize don't replace |
| Hero / banner section with image, heading, CTA button | Every Theme Store theme ships a customizable hero; Shopify reviewers check that text and image are editable | MEDIUM | Our hero is a two-column deal banner + "Just In" panel — both columns must be editor-configurable |
| Mobile-responsive layout at 320px–375px | Theme Store requirement; reviewers test on mobile viewport explicitly | MEDIUM | The editorial grid collapses to stacked single-column; format tabs scroll horizontally |
| Lazy-loading images | Core Web Vitals compliance; Shopify reviewers check LCP and CLS scores | LOW | Use `loading="lazy"` on all below-fold images; hero image must be `loading="eager"` |
| Skip-to-content link | WCAG 2.0 AA requirement; Theme Store fails themes without it | LOW | Hidden link that appears on keyboard focus; Dawn provides this by default |
| Keyboard navigation for all interactive elements | Theme Store accessibility requirement — all tabs, carousels, and modals must be keyboard-operable | MEDIUM | Format tab bar needs `role="tablist"`, arrow-key navigation |
| Focus visible styles (no outline: none) | WCAG 2.0 AA; Theme Store flags this | LOW | Never suppress `outline` globally; use `:focus-visible` for mouse suppression |
| Alt text on all images (or empty alt for decorative) | WCAG 2.0 AA; Theme Store audit tool checks for missing alt | LOW | Product card images should use product title as alt |
| Sufficient color contrast (4.5:1 text) | WCAG 2.0 AA; Theme Store color settings must produce passing contrast | MEDIUM | Dark theme palette likely passes but must be verified; theme editor color pickers need contrast warnings or defaults that pass |
| Working empty-state when collections have no products | Theme Store reviewers test with empty catalogs | LOW | "No products found" message in carousel sections |
| Section order controllable in theme editor | OS2.0 requirement — merchant must be able to add/remove/reorder sections on home page | LOW | JSON template plus proper `presets` in each section schema |

### Differentiators (Premium themes add these; free themes omit)

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Dual-panel hero (deal banner + featured grid) | Loris Lots' bulk pricing model needs prominent placement; more editorial than a single hero | HIGH | Two independently configurable columns in one section; editor settings for each panel's content |
| Format tab strip (DVDs / CDs / Blu-Ray / Vinyl) | Surfaces format-based navigation without a full menu click; very specific to media stores | MEDIUM | Tab bar driven by a linklist setting; active tab persists via URL param or sessionStorage |
| Format-filtered product carousel | Carousel content changes when format tab is selected; shows format-specific "Just Arrived" | HIGH | Requires JS to swap carousel content or multiple pre-rendered carousels shown/hidden |
| "Just Arrived" carousel with collection binding | Connects a section to a specific collection for automated freshness | MEDIUM | Section setting: `type: collection` pointing to a "new-arrivals" collection |
| Bulk discount progress hint | Contextual messaging about the discount threshold — unique to this store's model | MEDIUM | Reads cart subtotal via JS and renders a dynamic message; zero-cart state shows threshold only |
| Scrolling promo ticker with multiple messages | More dynamic than a static bar; common in premium themes | MEDIUM | CSS `marquee`-style animation or IntersectionObserver-based scroll; multiple text blocks via section blocks |
| "Collectible & Higher-End" featured product panel | Curated editorial slot for high-value items; gives the theme a magazine feel | MEDIUM | A configurable product-picker block inside the hero section |

### Anti-Features (Explicitly Exclude for v1.0)

| Anti-Feature | Why Requested | Why Problematic | Alternative |
|--------------|---------------|-----------------|-------------|
| AI search / floating search bar | Wireframe shows an AI search bubble prominently | Requires a third-party app (not native Shopify); breaks Theme Store requirement of no hard-coded external dependencies | Use Shopify's native Predictive Search API (`/search/suggest.json`) for a search bar; defer AI features to app integration |
| Left icon rail (Shop App navigation) | Wireframe shows a persistent left icon rail | Desktop icon rails are not a standard Shopify navigation pattern; would confuse merchants expecting a top nav; adds significant complexity for zero Theme Store benefit | Standard top navigation header; the rail is a client-specific UI pattern, not a theme pattern |
| Infinite scroll on home carousels | Feels fluid; reduces clicks | Performance penalty; poor accessibility (keyboard trap risk); difficult to maintain scroll position on back navigation | Horizontal scroll carousels with arrow buttons; "See all" links to collection |
| Hardcoded Loris Lots branding | Faster to ship | Theme Store requires themes to work for any merchant; hardcoded store name/copy fails review immediately | All text must be locale strings or schema settings |
| Real-time stock level display on product cards | Merchants want urgency signals | Requires AJAX polling or a Storefront API call per card; slows initial render; overkill for home page | Static "In stock" badge driven by `product.available`; real-time only on PDP |

---

## Page 2: Collection Page (Collection A — Pill Filters + Grid)

### Table Stakes (Theme Store will flag if missing)

| Feature | Why Expected / Required | Complexity | Notes |
|---------|------------------------|------------|-------|
| `main-collection-product-grid` section in JSON template | OS2.0 requirement — `templates/collection.json` must reference this section | LOW | Dawn ships this; must preserve it |
| Collection title and product count | Merchants expect this; reviewers check that collection metadata renders correctly | LOW | `{{ collection.title }}` and `{{ collection.products_count }}` |
| Pagination (numbered or "Load More") | Required — Shopify reviewers will test large collections; layout must not break at 100+ products | MEDIUM | `{% paginate %}` tag required; Theme Store accepts numbered, load-more, or infinite but all must be keyboard/screen-reader accessible |
| Sort-by dropdown | Theme Store requirement — merchants must be able to sort their collection | LOW | Use Shopify's native `collection.sort_options` to populate the dropdown; never hardcode options |
| Active filter state display | Shopify's storefront filters must show which filters are applied | MEDIUM | Active filters shown as removable chips — Shopify's Filter API returns `activeFilters` array |
| Filter by availability (In-Stock toggle) | Shopify's native storefront filters include availability; reviewers check filtering works | MEDIUM | Driven by `filter.storefront_filter` — no custom logic needed if using Shopify's filter API |
| Product card grid (image, title, price) | Core product browsing; reviewers verify all product card states | MEDIUM | Must handle: sale price, sold out, no image states; all must render correctly |
| Collection description (rich text) | Many merchants write collection descriptions for SEO; reviewers check it renders | LOW | Optional block in section schema; show/hide in editor |
| Empty collection state | Reviewers test with no products filtered; must show a helpful message, not a broken layout | LOW | Conditional `{% if collection.products.size == 0 %}` block |
| Mobile: filters accessible (drawer or collapsible) | Pill bar collapses on mobile — reviewers test filter UX on narrow viewport | MEDIUM | Filter pills horizontally scroll on mobile, or collapse into a "Filter" button opening a modal drawer |
| Breadcrumb navigation | Standard e-commerce UX; helps reviewers verify navigation hierarchy works | LOW | `Home / [collection title]` — implement as a simple Liquid snippet |
| Lazy-loading product images | Required for Core Web Vitals on large grids | LOW | `loading="lazy"` on all cards; first row can be `eager` |

### Differentiators (Premium themes add these; free themes omit)

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Horizontal pill filter bar (not sidebar) | Cleaner desktop UX; saves horizontal space for more product columns; matches wireframe Collection A | HIGH | Requires custom filter rendering using Shopify's `filter` objects rendered as pills; AJAX URL rewrite on selection |
| Sub-category chips (Genre: Action, Comedy, Drama…) | Deeper navigation within a collection without leaving the page; relevant to media (genre browsing) | HIGH | Implemented via tag-based collection URLs or a secondary linklist setting; must not break if tags don't exist |
| Live AJAX filtering (no page reload) | Premium UX; user sees results update in place | HIGH | Shopify's Section Rendering API — push new URL with `history.pushState`, fetch `?sections=main-collection-product-grid`, swap inner HTML |
| Product count per filter option | Shows "Action (47)" next to each filter pill — helps users understand filter density | MEDIUM | Available via Shopify's storefront filter API `filter.values[].count` |
| Column count setting in theme editor | Merchant can choose 4, 5, or 6 columns depending on their image style | LOW | A `select` setting in schema; CSS grid `grid-template-columns` driven by a CSS custom property |
| Results count display ("Showing 1–24 of 1,240") | Professional feel; helps users understand large catalogs | LOW | `{{ paginate.current_offset | plus: 1 }}–{{ paginate.current_offset | plus: paginate.page_size }}` |

### Anti-Features

| Anti-Feature | Why Requested | Why Problematic | Alternative |
|--------------|---------------|-----------------|-------------|
| Infinite scroll | Feels modern | Breaks browser back button; pagination is a Theme Store explicit recommendation for accessibility | "Load More" button or numbered pagination; keep URL-based navigation |
| Client-side filtering (JS array filter) | Simpler to implement than Shopify API | Doesn't work for large catalogs; ignores Shopify's product availability and filter counts | Use Shopify's storefront filter API + Section Rendering API |
| Sidebar filter layout | More filter space | Contradicts selected Collection A design; adds layout switching complexity | Pill bar with overflow-scroll on mobile; "All Filters" modal for advanced users is a v2 feature |

---

## Page 3: Product Detail Page (PDP A — Classic Gallery)

### Table Stakes (Theme Store will flag if missing)

| Feature | Why Expected / Required | Complexity | Notes |
|---------|------------------------|------------|-------|
| `main-product` section in JSON template | OS2.0 hard requirement — `templates/product.json` | LOW | Single most-reviewed template; every feature must be in this section |
| Product title, price, and description | Absolute minimum; reviewers will open a product and check these render | LOW | Price must handle: regular, compare-at (sale), and sold-out states |
| Variant selector (buttons or dropdowns) | Shopify themes must render all product options; reviewers test with 1-option, 2-option, and 3-option products | HIGH | For this theme: pill-style buttons per option (Format, Condition); must handle unavailable variant states (disabled/greyed out) |
| Add to Cart button | Core commerce action; reviewers test with all variant states | MEDIUM | Must be disabled when variant is unavailable or sold-out; must show loading state during AJAX add |
| Product image gallery with thumbnail nav | Standard for any PDP; reviewers check with products that have 1 image, multiple images, and no image | MEDIUM | Vertical thumbnail strip (left side) + main image (right) per PDP A wireframe |
| Variant image swap (gallery updates on variant change) | Shopify standard behaviour — selecting a variant should show its featured image | MEDIUM | `variant.featured_image` check; swap main gallery image via JS when variant selection changes |
| Breadcrumb navigation | Navigation hygiene; reviewers check context within store hierarchy | LOW | Home / Collection / Product Title |
| Sale price display (compare-at price strikethrough) | Required — reviewers will test with on-sale products | LOW | Conditional on `variant.compare_at_price > variant.price` |
| Sold-out state (button disabled, badge shown) | Required — reviewers test with all variants sold out | LOW | Conditional rendering; `cursor: not-allowed` on button |
| Quantity selector | Expected on every PDP; omitting it fails reviewer expectations | LOW | `<input type="number">` or stepper buttons; min="1" |
| Structured data (JSON-LD `Product` schema) | Google Rich Results; Theme Store reviewers check for SEO structured data | LOW | Dawn provides this in `snippets/structured-data.liquid`; verify it's included |
| Mobile sticky ATC bar | Strong merchant expectation; reviewers test mobile PDP UX | MEDIUM | Fixed-position bar at bottom of viewport on mobile; shows price and Add to Cart button; appears after scrolling past the inline ATC button |
| "You may also like" / related products | Shopify's `recommendations` API makes this easy; reviewers check that cross-sell is present | MEDIUM | `{% render 'product-recommendations' %}` using Shopify's product recommendations endpoint |
| Share buttons or share link | Present in most themes; reviewed as part of product page completeness | LOW | Native Web Share API (`navigator.share`) with clipboard fallback; no third-party SDK |
| Product media support (video, 3D model) | Theme Store requires support for all Shopify media types, not just images | HIGH | Shopify media object has `media_type` values: `image`, `video`, `external_video`, `model`; each needs a rendering branch |
| Accessibility: image alt text from media.alt | WCAG + Theme Store requirement | LOW | `{{ media.alt | escape }}` on every `<img>` |

### Differentiators (Premium themes add these; free themes omit)

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Pill-style variant selector (not native `<select>`) | Visual clarity for Format (DVD / Blu-Ray / Disc Only) — users scan available formats at a glance | MEDIUM | Custom pill buttons instead of `<select>`; unavailable combinations shown greyed with strikethrough |
| Bulk discount progress nudge on PDP | "Add $36.51 more for 38% off" — unique Loris Lots model; converts browsers into bulk buyers | HIGH | Reads current cart subtotal via `Shopify.cart` or a small fetch to `/cart.js`; dynamically computes remaining threshold; updates when ATC fires |
| Format/condition badge above title | Scannable metadata format — "DVD · Used · Like New" — common in media stores | LOW | A snippet driven by product type and a condition metafield or variant option |
| Sticky ATC sidebar (desktop) | Right info column stays in view while scrolling the long gallery and description | MEDIUM | CSS `position: sticky; top: var(--header-height)` on the right column; requires fixed header height CSS variable |
| Condition description in product description | "Disc condition: Like new, no scratches" — tailored messaging for used-goods buyers | LOW | This is content, not a theme feature — but the theme should support rich `product.description` rendering including inline HTML |
| Image zoom on hover or click | Premium UX; helps buyers inspect disc/cover condition | MEDIUM | CSS transform scale on hover (simple) or a lightbox modal (complex); avoid jQuery-based lightbox plugins |

### Anti-Features

| Anti-Feature | Why Requested | Why Problematic | Alternative |
|--------------|---------------|-----------------|-------------|
| Dynamic checkout buttons ("Buy it now") | Reduces friction | Requires Shopify Payments or an accelerated checkout provider; not available on all plans; Theme Store themes must not require specific payment providers | Standard ATC button only in v1.0; dynamic checkout button can be an optional section setting if Shopify's `content_for_additional_checkout_buttons` is used correctly |
| Inventory count display ("Only 3 left") | Urgency | Requires Storefront API or a liquid `product.variants[].inventory_quantity` — which Shopify no longer exposes directly in Liquid for security reasons | Use `product.variants[].available` boolean only; "In Stock" / "Sold Out" |
| Product reviews section (embedded) | Trust signals | Third-party app dependency; Theme Store themes cannot require apps | Build the review section as an app embed block placeholder using OS2.0 app blocks — the block renders nothing unless a reviews app installs itself |
| Tabbed description (Description / Details / Shipping tabs) | Organises long content | Hides content from Google (below-fold tabs); adds JS complexity | Expand/collapse accordion sections that are visible in DOM; or single scrollable description area |

---

## Page 4: Cart Drawer (Cart A — Right Slide-In)

### Table Stakes (Theme Store will flag if missing)

| Feature | Why Expected / Required | Complexity | Notes |
|---------|------------------------|------------|-------|
| Cart drawer as OS2.0 section | The drawer must be a proper `sections/cart-drawer.liquid` with schema, not a hardcoded snippet | MEDIUM | Dawn ships `cart-drawer.liquid` — use it as the base |
| Line item list with image, title, variant, price | Reviewers open the cart with multiple items; each item must show all attributes | LOW | `for item in cart.items` with `item.image`, `item.product.title`, `item.variant.title`, `item.final_price` |
| Quantity stepper (+ / − controls) per line item | Standard cart UX; reviewers test changing quantity | MEDIUM | Buttons that call `Shopify.updateItem()` or POST to `/cart/change.js`; optimistic UI update |
| Remove item per line item | Required — reviewer will test removing items | LOW | A remove button that calls `/cart/change.js` with `quantity: 0` |
| Subtotal display | Required — reviewers verify cart math is visible | LOW | `{{ cart.total_price | money }}` |
| Checkout CTA button | The primary action of any cart — reviewers click it | LOW | `href="/checkout"` — must open checkout, not a page within the theme |
| Drawer open/close animation | Required for usability review; a static overlay without animation fails the UX bar | MEDIUM | CSS transition on `transform: translateX(100%)` to `translateX(0)`; 200–300ms ease-out |
| Backdrop overlay with click-to-close | Standard modal/drawer UX; reviewers test closing the cart | LOW | Semi-transparent `<div>` behind the drawer; click fires drawer close |
| Trap focus within open drawer | WCAG 2.0 AA keyboard accessibility — focus must not escape the drawer when open | MEDIUM | JS focus trap: intercept Tab/Shift-Tab keys; Dawn's `focus-trap.js` utility handles this |
| Empty cart state | Reviewers test with empty cart; must show message and "Continue shopping" link | LOW | Conditional block: `{% if cart.item_count == 0 %}` |
| Cart count update in header icon | When item is added, the cart icon badge in the header must update without a page reload | MEDIUM | Dispatch a custom event (`cart:updated`) from the ATC handler; header listens and updates badge |
| ARIA: drawer role and labelling | `role="dialog"`, `aria-modal="true"`, `aria-label="Shopping cart"`, `aria-labelledby` | LOW | Required for screen reader accessibility — Theme Store checks ARIA on modal/dialog elements |
| `aria-live` region for quantity/total changes | Screen readers must announce cart updates | LOW | Wrap subtotal in `aria-live="polite"` |

### Differentiators (Premium themes add these; free themes omit)

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Bulk discount progress bar | Shows "$13.47 of $40 to unlock 38% off" with a visual progress bar — Loris Lots' core UX hook | HIGH | Requires knowing cart subtotal in real-time; progress bar updates after every quantity change; threshold value must be configurable in theme editor settings |
| Remaining-to-threshold counter ("$26.53 left") | Companion to the progress bar; converts cart review into a game — encourages adding one more item | MEDIUM | Computed in JS: `threshold - cart.total_price_cents / 100`; re-renders on cart update |
| Backdropblur on the page content behind the drawer | Premium visual effect; makes the drawer feel foregrounded and focused | LOW | `backdrop-filter: blur(2px)` on the overlay element — add a `@supports` check; no fallback needed for non-supporting browsers (graceful degradation) |
| Line item variant badge (DVD · Used) | Shows format and condition inline with each item — helps buyers review their mixed-format cart | LOW | `{{ item.variant.title }}` split on ` / ` to render as separate badge components |
| Animated item removal (slide out) | Premium feel; signals to users that the action worked | MEDIUM | CSS transition on max-height + opacity to 0 before removing from DOM |
| "Continue shopping" link that closes drawer (not navigation) | Better UX than a full page nav — buyer stays on current page | LOW | A `<button>` that fires the drawer close event instead of an `<a>` link |

### Anti-Features

| Anti-Feature | Why Requested | Why Problematic | Alternative |
|--------------|---------------|-----------------|-------------|
| Upsell / cross-sell product recommendations in cart drawer | Revenue maximization | Adds significant complexity; Shopify's product recommendations API is product-specific, not cart-aware; poor recommendations frustrate buyers | Defer to v2 when product recommendations can be tuned; the progress bar is a better conversion tool for this store's model |
| Discount code input in cart drawer | Convenience | Shopify's discount codes are applied at checkout, not in cart — a drawer discount field requires undocumented APIs or a redirect; misleads users if the code doesn't validate before checkout | Show a note: "Discount codes applied at checkout"; bulk discount is automatic so no code needed for the core use case |
| Free shipping progress bar (separate from bulk bar) | Common premium feature | Conflicts with Loris Lots' bulk discount bar — two progress bars would confuse buyers about which threshold to hit | One progress bar for the bulk discount threshold; the CTA copy can mention free shipping is included |
| Multi-currency switching in cart | International merchants want this | Requires Markets configuration; out of scope for v1.0 | Markets support is a future milestone once the theme is published |

---

## Feature Dependencies

```
[OS2.0 JSON Templates]
    └──required by──> [Theme Editor Settings for all sections]
                          └──required by──> [Theme Store submission]

[Shopify Filter API (storefront filters)]
    └──required by──> [Pill Filter Bar with live counts]
                          └──required by──> [AJAX live filtering]
                                               └──required by──> [Section Rendering API fetch]

[Variant selector (pill buttons)]
    └──required by──> [Variant image swap]
    └──required by──> [ATC button state (disabled/available)]
    └──required by──> [Bulk discount nudge on PDP] (needs selected variant price)

[Cart AJAX (/cart/add.js, /cart/change.js)]
    └──required by──> [Cart drawer quantity stepper]
    └──required by──> [Cart count badge update in header]
    └──required by──> [Bulk discount progress bar in drawer] (needs live cart total)

[Cart drawer section]
    └──required by──> [Focus trap (WCAG)]
    └──required by──> [ARIA dialog labelling]

[Product media (all media types)]
    └──required by──> [Gallery with thumbnail nav]
    └──required by──> [Variant image swap]

[Header section]
    └──required by──> [Cart count badge]
    └──required by──> [Drawer trigger button]

[Bulk discount progress bar (drawer)]
    ──enhances──> [Bulk discount nudge (PDP)]
    ──enhances──> [Promo ticker (home)]
    [All three are the same merchant value prop in different contexts]

[AJAX live filtering]
    ──conflicts with──> [Infinite scroll]
    [Both try to own the URL and product grid HTML simultaneously]

[Sticky ATC bar (mobile)]
    ──conflicts with──> [Fixed footer navigation]
    [Both claim the bottom of viewport; only one can exist]
```

### Dependency Notes

- **OS2.0 JSON templates require schema settings on every section:** Don't build any section without a `{% schema %}` block; reviewers test that every section's content can be changed in the theme editor. This is the single biggest cause of Theme Store rejection for developers coming from Debut-era themes.

- **Shopify Filter API requires storefront filters to be enabled on the collection:** The pill filter bar only works if the merchant has "Availability", "Price", "Product type", and tag/metafield filters configured in their Shopify admin. The theme must gracefully degrade to sort-only if no filters are configured.

- **Cart AJAX requires the `routes.cart_add_url` and `routes.cart_change_url` Liquid variables:** Never hardcode `/cart/add` — use Shopify's route variables for international compatibility.

- **Bulk discount progress bar depends on cart total in real-time:** The threshold value (e.g. $40) must be a theme editor setting, not hardcoded. The JS must re-fetch `/cart.js` after every add/remove to get the accurate subtotal.

- **Variant image swap depends on product media being tagged to variants:** Shopify only swaps the main gallery image if `variant.featured_media` is set — this requires merchants to assign images to variants in their admin. The theme cannot control this but should document it in the theme's help text.

---

## MVP Definition

### Launch With (v1.0 — Theme Store submission)

All items in Table Stakes for all four pages are required. The following differentiators are also in scope for v1.0 based on the PROJECT.md requirements:

- [ ] Dual-panel editorial hero (deal banner + featured grid) — core design identity
- [ ] Format tab strip with filtered "Just Arrived" carousel — core navigation model
- [ ] Pill filter bar on collection with AJAX live filtering — selected Collection A design
- [ ] Sub-category genre chips on collection — Collection A wireframe shows them
- [ ] Pill-style variant selector on PDP — Format/Condition selection is central to the product
- [ ] Bulk discount progress nudge on PDP — Loris Lots' conversion mechanism
- [ ] Sticky ATC bar on mobile PDP — wireframe B shows this; critical for mobile conversion
- [ ] Cart drawer with bulk progress bar — Cart A wireframe defines this explicitly
- [ ] Remaining-to-threshold counter in cart drawer — same wireframe

### Add After Validation (v1.x)

- [ ] Image zoom on hover (PDP) — adds complexity; not blocking Theme Store review
- [ ] Animated line item removal (cart drawer) — polish, not required
- [ ] "All Filters" modal for advanced filter options — complex; pill bar covers the primary use case
- [ ] Format-specific carousel swap on home (tab changes carousel content via AJAX) — nice but format tabs linking to collections is sufficient for v1.0

### Future Consideration (v2+)

- [ ] Upsell recommendations in cart drawer — requires product recommendations tuning
- [ ] AI/semantic search integration — app territory; out of Theme Store scope
- [ ] Multi-currency / Markets support
- [ ] Blog/article templates
- [ ] Search results page template
- [ ] Tabbed "Recently Viewed" section on home

---

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| OS2.0 JSON templates + schema for all sections | HIGH | LOW | P1 |
| Mobile responsiveness (all 4 pages) | HIGH | MEDIUM | P1 |
| WCAG 2.0 AA accessibility (focus, ARIA, alt text) | HIGH | MEDIUM | P1 |
| Core Web Vitals (lazy loading, no render-blocking JS) | HIGH | LOW | P1 |
| Variant selector (pill buttons) + image swap | HIGH | MEDIUM | P1 |
| Add to Cart AJAX + cart drawer open | HIGH | HIGH | P1 |
| Cart drawer (items, qty stepper, remove, checkout CTA) | HIGH | MEDIUM | P1 |
| Pill filter bar with live AJAX filtering | HIGH | HIGH | P1 |
| Editorial hero (deal banner + featured grid) | HIGH | MEDIUM | P1 |
| Format tab strip on home | HIGH | MEDIUM | P1 |
| Product recommendations ("You may also like") | MEDIUM | LOW | P1 |
| Bulk discount progress bar (cart drawer) | HIGH | HIGH | P1 |
| Bulk discount nudge on PDP | HIGH | MEDIUM | P1 |
| Sticky ATC bar (mobile PDP) | HIGH | MEDIUM | P1 |
| Sub-category genre chips (collection) | MEDIUM | MEDIUM | P2 |
| Image zoom on PDP | MEDIUM | MEDIUM | P2 |
| Scrolling promo ticker | LOW | MEDIUM | P2 |
| Animated item removal (cart) | LOW | MEDIUM | P3 |
| Upsell in cart drawer | MEDIUM | HIGH | P3 |

**Priority key:**
- P1: Must have for v1.0 Theme Store submission
- P2: Should have; add if time allows in v1.0, else v1.1
- P3: Future milestone only

---

## Competitor Feature Analysis (Theme Store Premium Themes)

| Feature | Prestige (free) | Dawn (free) | Impulse / Flex (paid $300+) | Our Approach |
|---------|----------------|-------------|------------------------------|--------------|
| Hero section | Single image/text | Slideshow | Video + image + editorial | Two-column editorial deal banner |
| Collection filters | Sidebar toggle | Sidebar + horizontal | Horizontal pills, AJAX | Horizontal pills, AJAX |
| PDP variant selector | Native `<select>` | Pill buttons (limited) | Pill buttons, swatch images | Pill buttons (Format + Condition) |
| Cart | Full-page cart | Drawer | Drawer with upsell | Drawer with bulk progress bar |
| Sticky ATC | No | No | Yes (desktop + mobile) | Yes (mobile only in v1.0) |
| Product recommendations | No | Yes (basic) | Yes with filtering | Yes (Shopify API, same collection) |
| Animation quality | Minimal | Minimal | Rich transitions | CSS transitions only; no heavy JS |
| Bulk discount logic | None | None | None | Custom (unique to this theme) |

---

## Theme Store Review: Specific Pass/Fail Criteria

Based on Shopify's published review requirements (stable since OS2.0, August 2022):

**Hard failures (theme is rejected, not conditionally approved):**
- Templates that use `.liquid` extension instead of `.json` for OS2.0 templates (`index.json`, `collection.json`, `product.json`, `cart.json`, `page.json`)
- Hardcoded store-specific content (Loris Lots name, specific prices, client URLs) in any section or snippet
- Missing `{% schema %}` on sections that have customizable content
- External scripts loaded via `<script src="...">` from third-party CDNs without a settings toggle
- Missing focus styles (any `outline: none` or `outline: 0` without `:focus-visible` alternative)
- Colour contrast failures below 4.5:1 for body text
- Cart/checkout flow that does not reach Shopify checkout (e.g., redirecting to an external checkout)
- Missing alt text on product images (empty `alt=""` is acceptable only for decorative images)
- JavaScript errors in the browser console on any core user flow (home → collection → PDP → add to cart → checkout)

**Common conditional approvals (fix-and-resubmit):**
- Sections that exist but have no schema settings (fixed: add schema)
- Product media types not handled (fixed: add video/3D rendering branches)
- Mobile layout issues at 375px (fixed: responsive adjustments)
- Missing empty states for collections, cart, search results

---

## Sources

- Wireframe files directly inspected: `/wireframes/home-screens.jsx`, `/wireframes/collection-screens.jsx`, `/wireframes/product-screens.jsx`, `/wireframes/cart-screens.jsx`
- Wireframe screenshots inspected: all 15 screenshots in `/wireframes/screenshots/`
- PROJECT.md and STATE.md: `/Users/henrygillard/Downloads/theme_export__loris-lots-com-debut__30APR2026-0409pm/.planning/`
- Existing Debut codebase: `sections/` and `snippets/` directories examined for baseline understanding
- Shopify Theme Store requirements: training data through August 2025; stable since OS2.0 rollout (2021–2022); verify at https://shopify.dev/docs/storefronts/themes/store/requirements before submission
- Dawn reference theme feature set: training data; verify current Dawn feature set at https://github.com/Shopify/dawn

---

*Feature research for: Shopify OS2.0 Theme — Loris Lots*
*Researched: 2026-05-06*
