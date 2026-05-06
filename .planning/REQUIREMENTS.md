# Requirements: Loris Lots — Shopify Custom Theme

**Defined:** 2026-05-06
**Core Value:** A merchant installs this theme and has a polished used-media storefront that passes Shopify Theme Store review — zero custom development needed.

## v1 Requirements

### Scaffold & Design System

- [ ] **SCAFFOLD-01**: Developer can fork Dawn, configure settings_schema.json with theme_info metadata, and develop locally using Shopify CLI 3 (`shopify theme dev`)
- [ ] **SCAFFOLD-02**: Merchant can set brand colors and typography in the Shopify theme editor; values flow through CSS custom properties (`--ll-color-*`, `--ll-font-*`) to all sections via layout/theme.liquid
- [ ] **SCAFFOLD-03**: All Debut artifacts are removed — lazysizes.js, `{% include %}` (replaced by `{% render %}`), client-specific conditionals, hardcoded hex colors, console.log calls, and external image URLs; Dawn JSON templates replace all legacy `.liquid` templates
- [ ] **SCAFFOLD-04**: All interactive components use a consistent Vanilla JS class-per-component pattern with `data-ll-*` attributes and communicate via CustomEvent bus (`ll:cart:open`, `ll:cart:updated`)

### Shared Components

- [ ] **SHARED-01**: Product card snippet (`ll-product-card`) renders image, title, price, format badge, condition badge, and ATC button; used consistently across carousels, grids, and related products rails
- [ ] **SHARED-02**: Bulk discount progress snippet (`ll-bulk-progress`) renders a configurable progress bar and threshold message; reused in cart drawer, PDP nudge, and home promo ticker — built once, included everywhere
- [ ] **SHARED-03**: Header displays logo, search, cart icon with live item-count badge, and main navigation; footer displays configurable link columns and social links; both fully configurable in theme editor
- [ ] **SHARED-04**: Scrolling announcement bar displays configurable deal messages and marquee speed via theme editor settings; renders on all pages via layout/theme.liquid

### Cart Drawer

- [ ] **CART-01**: User can open the cart drawer from any page via ATC buttons or the cart icon; drawer slides in from the right with a smooth CSS transition and a dimmed overlay backdrop
- [ ] **CART-02**: User can increment, decrement, or remove line items in the cart drawer; totals update in real time via the `/cart.js` Ajax API; a request queue prevents race conditions from rapid interactions
- [ ] **CART-03**: User sees a bulk discount progress bar in the cart drawer showing progress toward the discount threshold; threshold amount and message text are configurable in theme editor settings
- [ ] **CART-04**: Cart drawer traps keyboard focus when open (`role="dialog"`, `aria-modal="true"`), closes on Esc key or backdrop click, and returns focus to the triggering element on close

### Collection Page

- [ ] **COL-01**: User can filter products by Format, Condition, Genre, and Price using horizontal pill/chip filters that live-reload the product grid via Shopify Section Rendering API without a full page reload
- [ ] **COL-02**: Collection page displays a responsive product grid using `ll-product-card` with a sort-by dropdown and total results count
- [ ] **COL-03**: Active filters display as individually removable chips with a clear-all button; removing any filter live-reloads the grid and updates the URL via `history.pushState`
- [ ] **COL-04**: Collection page supports pagination (load-more button or numbered pages) that functions correctly with active filter state

### Product Detail Page

- [ ] **PDP-01**: User can view product images in a thumbnail gallery with main image swap on click; gallery handles Shopify video and 3D model media types in addition to images
- [ ] **PDP-02**: User can select product format (DVD / Blu-Ray / CD / Vinyl) and condition (New / Used / Like New) using pill-style variant selectors; price and availability update on selection without a page reload
- [ ] **PDP-03**: On mobile, a sticky ATC bar appears at the bottom of the screen when the inline ATC button scrolls out of view (via IntersectionObserver); an inline bulk discount nudge message updates live when cart contents change
- [ ] **PDP-04**: Related products are displayed in a horizontal-scroll rail below the fold, populated via Shopify's product recommendations API

### Home Page

- [ ] **HOME-01**: Home page displays a full-width editorial hero section with a deal message (e.g. "Add $40, get 38% off at checkout"), a CTA button, and fully configurable text and colors in the theme editor
- [ ] **HOME-02**: A format tab strip (DVDs / CDs / Blu-Ray / Vinyl) displays below the hero and links to the corresponding collection pages; tabs are configurable via a link list in the theme editor
- [ ] **HOME-03**: Home page includes configurable product carousel sections (e.g. Just Arrived, Editor's Picks) with horizontal scroll using `ll-product-card`; collection source is configurable per carousel in theme editor
- [ ] **HOME-04**: Scrolling announcement bar renders on all pages with configurable messages and speed (see SHARED-04)

### Theme Store Compliance

- [ ] **COMPLY-01**: All pages pass WCAG 2.0 AA requirements: keyboard navigation, visible skip link, correct focus management, ARIA labels on all interactive elements, and sufficient color contrast
- [ ] **COMPLY-02**: All 4 page types achieve passing Core Web Vitals scores (LCP, CLS, INP) via native image lazy loading, `fetchpriority="high"` on hero image, and deferred JS loading
- [ ] **COMPLY-03**: All section and block settings function correctly in the Shopify theme editor with no broken defaults; all section presets render cleanly on a fresh store
- [ ] **COMPLY-04**: Theme passes `shopify theme check` with zero errors, includes all required locale files, has complete `theme_info` metadata in settings_schema.json, and produces a valid `.zip` via `shopify theme package`

## v2 Requirements

### Extended Pages

- **PAGE-01**: Search results page with pill filters and grid layout
- **PAGE-02**: Blog / article templates matching the editorial design language
- **PAGE-03**: Custom 404 page with navigation and product suggestions

### Cart Enhancements

- **CART-05**: Upsell / cross-sell product recommendations in cart drawer
- **CART-06**: Save for later functionality on line items

### Collection Enhancements

- **COL-05**: Infinite scroll as an alternative to pagination
- **COL-06**: Quick-view product overlay from collection grid

### Advanced Features

- **ADV-01**: AI / predictive search overlay (Shopify Predictive Search API)
- **ADV-02**: Multi-currency display support
- **ADV-03**: Wishlisting / save product (requires app integration)
- **ADV-04**: Recently viewed products rail

## Out of Scope

| Feature | Reason |
|---------|--------|
| Hydrogen / headless storefront | Not eligible for Shopify Theme Store; different deployment model entirely |
| Alpine.js or any JS framework | Stack decision: Vanilla JS only for zero-dependency Theme Store compliance |
| Multiple layout variants per page | Scope control for v1.0; one strong opinion per page is better theme design |
| Left icon rail (from wireframes) | Anti-feature for Theme Store — non-portable, store-specific nav pattern |
| AI search floating bubble (from wireframes) | Replaced by native Shopify Predictive Search in a standard search input |
| Checkout customization | Requires Shopify Plus; outside theme scope |
| App development | Theme only — no custom Shopify apps |
| Shopify Functions / discount logic | The bulk discount is a Shopify Script/Function; theme reads the threshold, doesn't apply it |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| SCAFFOLD-01 | Phase 1 | Pending |
| SCAFFOLD-02 | Phase 1 | Pending |
| SCAFFOLD-03 | Phase 1 | Pending |
| SCAFFOLD-04 | Phase 1 | Pending |
| SHARED-01 | Phase 2 | Pending |
| SHARED-02 | Phase 2 | Pending |
| SHARED-03 | Phase 2 | Pending |
| SHARED-04 | Phase 2 | Pending |
| CART-01 | Phase 2 | Pending |
| CART-02 | Phase 2 | Pending |
| CART-03 | Phase 2 | Pending |
| CART-04 | Phase 2 | Pending |
| HOME-01 | Phase 3 | Pending |
| HOME-02 | Phase 3 | Pending |
| HOME-03 | Phase 3 | Pending |
| HOME-04 | Phase 3 | Pending |
| COL-01 | Phase 4 | Pending |
| COL-02 | Phase 4 | Pending |
| COL-03 | Phase 4 | Pending |
| COL-04 | Phase 4 | Pending |
| PDP-01 | Phase 5 | Pending |
| PDP-02 | Phase 5 | Pending |
| PDP-03 | Phase 5 | Pending |
| PDP-04 | Phase 5 | Pending |
| COMPLY-01 | Phase 6 | Pending |
| COMPLY-02 | Phase 6 | Pending |
| COMPLY-03 | Phase 6 | Pending |
| COMPLY-04 | Phase 6 | Pending |

**Coverage:**
- v1 requirements: 28 total
- Mapped to phases: 28
- Unmapped: 0 ✓

---
*Requirements defined: 2026-05-06*
*Last updated: 2026-05-06 — traceability filled by roadmapper*
