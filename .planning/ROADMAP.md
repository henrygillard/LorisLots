# Roadmap: Loris Lots — Shopify Custom Theme

## Overview

Six phases deliver a Dawn-based Shopify OS2.0 theme from scratch: scaffold and design tokens first, shared components and cart drawer second (they are dependencies for everything else), then the three page types in ascending complexity order (Home, Collection, PDP), and finally a dedicated compliance pass to satisfy Shopify Theme Store review requirements.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Scaffold & Design System** - Fork Dawn, purge Debut artifacts, wire CSS custom properties and JS component pattern
- [ ] **Phase 2: Shared Components & Cart Drawer** - Build ll-product-card, ll-bulk-progress, header/footer, announcement bar, and cart drawer
- [ ] **Phase 3: Home Page** - Implement editorial hero, format tab strip, and product carousel sections
- [ ] **Phase 4: Collection Page** - Implement pill filter bar with AJAX live-reload, product grid, active filter chips, and pagination
- [ ] **Phase 5: Product Detail Page** - Implement thumbnail gallery, pill variant selector, sticky ATC bar, and related products rail
- [ ] **Phase 6: Theme Store Compliance** - Accessibility audit, Core Web Vitals pass, theme editor QA, theme check zero errors, and packaging

## Phase Details

### Phase 1: Scaffold & Design System
**Goal**: A clean Dawn-based project is running locally with all Debut artifacts removed, brand tokens flowing through CSS custom properties, and a consistent Vanilla JS component pattern ready for every section to use
**Depends on**: Nothing (first phase)
**Requirements**: SCAFFOLD-01, SCAFFOLD-02, SCAFFOLD-03, SCAFFOLD-04
**Success Criteria** (what must be TRUE):
  1. Running `shopify theme dev` against the repository starts a local preview with no errors in the terminal or browser console
  2. Changing brand colors and typography in the Shopify theme editor immediately updates the live preview on all pages without a code change
  3. No Debut-era artifacts are present: `{% include %}` tags, lazysizes.js, hardcoded hex colors, console.log calls, and external image URLs are all absent from the codebase
  4. Every interactive element in the theme uses a `data-ll-*` attribute selector and communicates via the `ll:` CustomEvent bus — no ad-hoc inline event handlers or global variable coupling
**Plans**: TBD
**UI hint**: yes

### Phase 2: Shared Components & Cart Drawer
**Goal**: The product card snippet, bulk progress bar, site header and footer, announcement bar, and cart drawer all exist and work — providing the building blocks every subsequent page phase assembles
**Depends on**: Phase 1
**Requirements**: SHARED-01, SHARED-02, SHARED-03, SHARED-04, CART-01, CART-02, CART-03, CART-04
**Success Criteria** (what must be TRUE):
  1. A product card placed in any context (carousel, grid, or rail) renders image, title, price, format badge, condition badge, and ATC button correctly
  2. Clicking any ATC button or the cart icon opens the cart drawer with a smooth slide-in animation and a dimmed backdrop; clicking the backdrop or pressing Esc closes it and returns focus to the triggering element
  3. Incrementing, decrementing, or removing a line item in the open drawer updates totals in real time with no duplicate requests, even when interactions happen rapidly
  4. The bulk discount progress bar in the drawer reflects correct progress toward the configured threshold, and the threshold amount and message are editable in the theme editor
  5. The scrolling announcement bar renders on every page with configurable messages and marquee speed set via theme editor
**Plans**: TBD
**UI hint**: yes

### Phase 3: Home Page
**Goal**: The editorial home page is fully assembled — hero deal banner, format tab strip, and configurable product carousels — and every element is adjustable via the Shopify theme editor
**Depends on**: Phase 2
**Requirements**: HOME-01, HOME-02, HOME-03, HOME-04
**Success Criteria** (what must be TRUE):
  1. The home page displays a full-width hero with a deal message, CTA button, and background; editing text and colors in the theme editor updates the preview instantly
  2. A format tab strip (DVDs, CDs, Blu-Ray, Vinyl) appears below the hero; each tab links to its collection page; tab labels and links are editable via a link list in the theme editor
  3. At least two product carousel sections (e.g., Just Arrived, Editor's Picks) display with horizontal scroll, each sourcing products from a configurable collection
  4. The announcement bar (built in Phase 2) renders correctly on the home page with no layout breakage at 375px viewport width
**Plans**: TBD
**UI hint**: yes

### Phase 4: Collection Page
**Goal**: The collection page delivers live pill-filter browsing — users can filter by format, condition, genre, and price; the grid updates without a full page reload; active filters are individually removable; and pagination works alongside any filter state
**Depends on**: Phase 2
**Requirements**: COL-01, COL-02, COL-03, COL-04
**Success Criteria** (what must be TRUE):
  1. Clicking a pill filter (Format, Condition, Genre, or Price) live-reloads only the product grid via the Section Rendering API — the rest of the page does not reload and the URL updates via history.pushState
  2. Active filters appear as individually removable chips above the grid; clicking the X on any chip removes that filter and re-renders the grid; a clear-all button removes all filters at once
  3. The product grid uses ll-product-card and displays a sort-by dropdown and a total results count that updates after every filter change
  4. Pagination (load-more or numbered pages) works correctly when filters are active — navigating pages does not reset the active filter state
**Plans**: TBD
**UI hint**: yes

### Phase 5: Product Detail Page
**Goal**: The PDP presents a complete product browsing and purchase experience — image gallery with media type support, pill variant selectors for format and condition, a sticky ATC bar on mobile, and a related products rail
**Depends on**: Phase 2
**Requirements**: PDP-01, PDP-02, PDP-03, PDP-04
**Success Criteria** (what must be TRUE):
  1. Clicking a thumbnail in the gallery swaps the main image without a page reload; Shopify video and 3D model media appear in the gallery alongside images
  2. Selecting a format (DVD, Blu-Ray, CD, Vinyl) or condition (New, Used, Like New) pill updates the displayed price and availability in real time without a page reload
  3. On a mobile viewport (375px), scrolling past the inline ATC button causes a sticky ATC bar to appear at the bottom of the screen; the bar disappears when the inline button scrolls back into view
  4. A related products rail appears below the fold populated via Shopify's recommendations API; the rail is horizontally scrollable on mobile
**Plans**: TBD
**UI hint**: yes

### Phase 6: Theme Store Compliance
**Goal**: The finished theme passes Shopify Theme Store review: all pages are keyboard-navigable and WCAG 2.0 AA compliant, Core Web Vitals are in passing range, every theme editor setting works on a fresh store, shopify theme check reports zero errors, and a valid theme package is produced
**Depends on**: Phase 5
**Requirements**: COMPLY-01, COMPLY-02, COMPLY-03, COMPLY-04
**Success Criteria** (what must be TRUE):
  1. Every interactive element on all four page types is reachable and operable by keyboard alone; a visible skip link appears on focus; ARIA labels and roles are present on all interactive elements; color contrast passes WCAG 2.0 AA
  2. A Lighthouse report run against the live theme (not local dev) shows passing Core Web Vitals (LCP, CLS, INP) on at least the home and collection pages
  3. Installing the theme on a fresh Shopify development store and adding section presets via the theme editor produces a complete, unbroken storefront with no missing defaults or broken settings
  4. Running `shopify theme check` against the repository produces zero errors, all required locale keys are present, theme_info metadata is complete, and `shopify theme package` produces a valid .zip without warnings

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5 → 6

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Scaffold & Design System | 0/TBD | Not started | - |
| 2. Shared Components & Cart Drawer | 0/TBD | Not started | - |
| 3. Home Page | 0/TBD | Not started | - |
| 4. Collection Page | 0/TBD | Not started | - |
| 5. Product Detail Page | 0/TBD | Not started | - |
| 6. Theme Store Compliance | 0/TBD | Not started | - |
