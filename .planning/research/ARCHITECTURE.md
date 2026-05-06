# Architecture Research

**Domain:** Shopify OS2.0 theme (Dawn base) — used media storefront
**Researched:** 2026-05-06
**Confidence:** HIGH — based on Shopify's published OS2.0 architecture, Dawn's known file structure, and direct inspection of the existing Debut codebase and wireframes.

---

## Standard Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    BROWSER / THEME EDITOR                        │
├─────────────────────────────────────────────────────────────────┤
│  layout/theme.liquid  ←  single root document wrapper            │
│  Injects: CSS tokens, global JS, cart-drawer HTML               │
├─────────────────────────────────────────────────────────────────┤
│  templates/*.json     ←  OS2.0 JSON templates (section lists)   │
│  Each page type = ordered list of section references             │
│  templates/index.json / collection.json / product.json /        │
│  cart.json                                                       │
├─────────────────────────────────────────────────────────────────┤
│  sections/*.liquid    ←  self-contained page regions             │
│  Each has: HTML + {% schema %} + scoped <style> + <script>      │
│                                                                  │
│  NEW sections (create from scratch):                            │
│  ├── sections/ll-announcement-bar.liquid                        │
│  ├── sections/ll-header.liquid                                  │
│  ├── sections/ll-home-hero.liquid                               │
│  ├── sections/ll-format-tabs.liquid                             │
│  ├── sections/ll-product-carousel.liquid                        │
│  ├── sections/ll-collection-banner.liquid                       │
│  ├── sections/ll-collection-filter-bar.liquid                   │
│  ├── sections/ll-main-collection-grid.liquid                    │
│  ├── sections/ll-main-product.liquid                            │
│  ├── sections/ll-related-products.liquid                        │
│  ├── sections/ll-cart-drawer.liquid                             │
│  └── sections/ll-footer.liquid                                  │
│                                                                  │
│  MODIFIED Dawn sections (adapt, do not wholesale replace):      │
│  └── (none — clean-room build on Dawn scaffold, not patching    │
│       Debut sections into Dawn)                                  │
├─────────────────────────────────────────────────────────────────┤
│  snippets/*.liquid    ←  reusable partials (no schema)          │
│  ├── snippets/ll-product-card.liquid                            │
│  ├── snippets/ll-pill.liquid                                    │
│  ├── snippets/ll-format-badge.liquid                            │
│  ├── snippets/ll-price.liquid                                   │
│  ├── snippets/ll-bulk-progress.liquid                           │
│  └── snippets/ll-icon-*.liquid                                  │
├─────────────────────────────────────────────────────────────────┤
│  assets/              ←  flat file store (no subfolders)        │
│  ├── ll-theme.css               CSS custom properties (tokens)  │
│  ├── ll-cart-drawer.js          Cart API + drawer open/close    │
│  ├── ll-filter-bar.js           Pill filter URL + DOM update    │
│  ├── ll-product-gallery.js      Thumbnail→main image swap       │
│  ├── ll-variant-selector.js     Variant pill → ATC price sync   │
│  └── ll-global.js               Shared utilities, aria helpers  │
├─────────────────────────────────────────────────────────────────┤
│  config/                                                         │
│  ├── settings_schema.json       Global theme editor settings    │
│  └── settings_data.json         Saved merchant settings         │
└─────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Implementation |
|-----------|---------------|----------------|
| `layout/theme.liquid` | Root HTML shell, CSS/JS inclusion, cart-drawer mount point | Liquid — global scope |
| `templates/*.json` | Declare which sections appear on each page type and their default order | JSON — OS2.0 only |
| `sections/ll-*.liquid` | Self-contained page regions with merchant-configurable schema | Liquid + schema JSON |
| `snippets/ll-*.liquid` | Shared partials (product card, price, badges) with no schema | Liquid only |
| `assets/ll-*.js` | Vanilla JS interaction modules — one file per concern | ES module IIFE or class |
| `assets/ll-theme.css` | CSS custom property definitions consumed by all sections | Plain CSS |
| `config/settings_schema.json` | Theme editor global settings (brand colors, fonts, layout options) | JSON schema |

---

## Recommended Project Structure

```
Dawn scaffold root/
├── layout/
│   └── theme.liquid              MODIFY — add cart-drawer mount point,
│                                 ll-theme.css link, global JS includes
│
├── templates/
│   ├── index.json                CREATE — home page section list
│   ├── collection.json           CREATE — collection page section list
│   ├── product.json              CREATE — PDP section list
│   └── cart.json                 CREATE — cart page (minimal; drawer is
│                                 in layout, not cart template)
│
├── sections/
│   ├── ll-announcement-bar.liquid   CREATE — scrolling promo ticker
│   ├── ll-header.liquid             CREATE — logo + nav + cart icon trigger
│   ├── ll-home-hero.liquid          CREATE — deal banner + "Just In" panel
│   ├── ll-format-tabs.liquid        CREATE — DVDs/CDs/Blu-Ray/Vinyl chips
│   ├── ll-product-carousel.liquid   CREATE — "Just Arrived" horizontal row
│   ├── ll-collection-banner.liquid  CREATE — collection title + meta line
│   ├── ll-collection-filter-bar.liquid  CREATE — pill filters + sort pill
│   ├── ll-main-collection-grid.liquid   CREATE — paginated product grid
│   ├── ll-main-product.liquid       CREATE — gallery + variants + ATC
│   ├── ll-related-products.liquid   CREATE — "You may also like" carousel
│   ├── ll-cart-drawer.liquid        CREATE — right slide-in drawer (global)
│   └── ll-footer.liquid             CREATE — footer links + legal
│
├── snippets/
│   ├── ll-product-card.liquid    CREATE — used by carousel + grid
│   ├── ll-pill.liquid            CREATE — filter/format pill component
│   ├── ll-format-badge.liquid    CREATE — "DVD · Used" badge
│   ├── ll-price.liquid           CREATE — price display with sale handling
│   ├── ll-bulk-progress.liquid   CREATE — "$X to unlock 38% off" bar
│   └── ll-icon-*.liquid          CREATE — SVG icons (cart, close, chevron…)
│
├── assets/
│   ├── ll-theme.css              CREATE — CSS custom properties (design tokens)
│   ├── ll-global.js              CREATE — shared utilities, aria helpers
│   ├── ll-cart-drawer.js         CREATE — cart AJAX + drawer open/close
│   ├── ll-filter-bar.js          CREATE — pill filter URL manipulation
│   ├── ll-product-gallery.js     CREATE — thumbnail gallery interaction
│   └── ll-variant-selector.js    CREATE — variant → price/ATC sync
│
└── config/
    ├── settings_schema.json      CREATE — brand color, font, layout settings
    └── settings_data.json        CREATE — default values
```

### Structure Rationale

- **`ll-` prefix on all custom files:** Prevents name collisions when Dawn is cloned as scaffold and its built-in sections/snippets are pruned. Clear provenance during Theme Store review.
- **One JS file per concern:** Cart drawer JS is independently loadable before the drawer section renders. Filter bar JS can be excluded on non-collection pages. No bundler required — each file is included via `asset_url` only on pages that need it.
- **Snippets for repeated partials:** Product card is used in carousels, grids, and the related-products rail. Defining it once in a snippet and calling `{% render 'll-product-card', product: product %}` everywhere avoids drift.
- **CSS custom properties in one file:** `ll-theme.css` defines all `--ll-*` tokens. Sections reference tokens, not raw hex values. Merchant color changes in the theme editor write to `settings.color_*`; a `<style>` block in `layout/theme.liquid` rewrites the CSS custom properties to match.

---

## Architectural Patterns

### Pattern 1: OS2.0 JSON Template + Section Schema

**What:** Every page type has a `templates/*.json` file that lists sections by ID. Each section's Liquid file contains a `{% schema %}` block defining its editor settings, blocks, and presets. The theme editor reads these schemas to present controls.

**When to use:** All pages. This is mandatory for OS2.0 compliance and Theme Store eligibility.

**Trade-offs:** More verbose than legacy Liquid templates; pays off with section-everywhere editor flexibility and Theme Store approval.

**Example — `templates/index.json`:**
```json
{
  "sections": {
    "announcement-bar": { "type": "ll-announcement-bar", "settings": {} },
    "header": { "type": "ll-header", "settings": {} },
    "home-hero": { "type": "ll-home-hero", "settings": {
      "deal_heading": "Add $40, get 38% off at checkout.",
      "deal_cta_label": "Shop the lots",
      "deal_cta_url": "/collections/all"
    }},
    "format-tabs": { "type": "ll-format-tabs", "settings": {} },
    "just-arrived": { "type": "ll-product-carousel", "settings": {
      "title": "Just Arrived",
      "collection": "just-arrived",
      "products_to_show": 6
    }},
    "footer": { "type": "ll-footer", "settings": {} }
  },
  "order": [
    "announcement-bar", "header", "home-hero",
    "format-tabs", "just-arrived", "footer"
  ]
}
```

**Example — section schema block (inside `ll-home-hero.liquid`):**
```liquid
{% schema %}
{
  "name": "Home Hero",
  "tag": "section",
  "class": "ll-home-hero",
  "settings": [
    {
      "type": "text",
      "id": "deal_heading",
      "label": "Deal heading",
      "default": "Add $40, get 38% off at checkout."
    },
    {
      "type": "url",
      "id": "deal_cta_url",
      "label": "CTA link"
    },
    {
      "type": "text",
      "id": "deal_cta_label",
      "label": "CTA button text",
      "default": "Shop the lots"
    }
  ],
  "blocks": [
    {
      "type": "featured_collection_card",
      "name": "Featured collection card",
      "limit": 3,
      "settings": [
        { "type": "collection", "id": "collection", "label": "Collection" },
        { "type": "text", "id": "label", "label": "Label", "default": "JUST IN" }
      ]
    }
  ],
  "presets": [{ "name": "Home Hero" }]
}
{% endschema %}
```

---

### Pattern 2: Global Cart Drawer in `layout/theme.liquid`

**What:** The cart drawer (`ll-cart-drawer.liquid`) is included once in the layout, not in the cart template. It is always present in the DOM, hidden by default. Any "Add to cart" button anywhere on the site can trigger it open. The cart template (`cart.json`) either redirects to drawer or shows a minimal fallback page.

**When to use:** Cart A (right drawer) variant. This is the standard Dawn approach for cart drawers.

**Trade-offs:** Drawer HTML is loaded on every page (small payload); avoids page redirect on ATC. No-JS fallback requires the cart template page to still exist.

**Integration in `layout/theme.liquid`:**
```liquid
{%- render 'll-cart-drawer' -%}

{%- comment -%} Token injection from theme settings {%- endcomment -%}
<style>
  :root {
    --ll-color-ink: {{ settings.color_ink }};
    --ll-color-bg: {{ settings.color_bg }};
    --ll-color-accent: {{ settings.color_accent }};
    --ll-color-surface: {{ settings.color_surface }};
    --ll-font-body: {{ settings.type_body_font.family }}, {{ settings.type_body_font.fallback_families }};
  }
</style>
```

---

### Pattern 3: Vanilla JS — Class-per-Component with `data-` Attributes

**What:** Each interactive UI component is a JS class that queries `[data-ll-*]` attributes, registers event listeners on mount, and communicates via `CustomEvent` on `document`. No shared global state object. No framework.

**When to use:** All JS interactions in this theme. Chosen over module pattern (less discoverable) and web components (CustomElementRegistry overkill for Theme Store, Safari compatibility concerns pre-2023).

**Trade-offs:** Slightly more boilerplate than Alpine.js; zero runtime dependency. Passes Theme Store JS review. Each class is self-contained and tree-shakes cleanly.

**Example — cart drawer:**
```javascript
// assets/ll-cart-drawer.js

class LLCartDrawer {
  constructor() {
    this.drawer = document.querySelector('[data-ll-cart-drawer]');
    this.overlay = document.querySelector('[data-ll-cart-overlay]');
    this.closeBtn = this.drawer?.querySelector('[data-ll-cart-close]');
    this._bindEvents();
  }

  open() {
    this.drawer.setAttribute('aria-hidden', 'false');
    this.drawer.classList.add('is-open');
    this.overlay.classList.add('is-visible');
    document.body.style.overflow = 'hidden';
    this.refresh();
  }

  close() {
    this.drawer.setAttribute('aria-hidden', 'true');
    this.drawer.classList.remove('is-open');
    this.overlay.classList.remove('is-visible');
    document.body.style.overflow = '';
  }

  async refresh() {
    const res = await fetch('/cart.js');
    const cart = await res.json();
    document.dispatchEvent(new CustomEvent('ll:cart:updated', { detail: cart }));
    this._renderItems(cart);
  }

  _bindEvents() {
    document.addEventListener('ll:cart:open', () => this.open());
    this.closeBtn?.addEventListener('click', () => this.close());
    this.overlay?.addEventListener('click', () => this.close());
  }

  _renderItems(cart) { /* update drawer DOM with cart.items */ }
}

document.addEventListener('DOMContentLoaded', () => {
  window.llCartDrawer = new LLCartDrawer();
});
```

**ATC buttons fire the event:**
```javascript
// Inside ll-variant-selector.js or any ATC button handler:
document.dispatchEvent(new CustomEvent('ll:cart:open'));
```

---

### Pattern 4: Shopify Storefront Filters API for Pill Filters

**What:** Collection A uses pill filters (sort, condition, genre subcategory). In Dawn/OS2.0, these are implemented via Shopify's native storefront filtering (`collection.filters`, `filter.values`, URL parameter `filter.p.m.*`). The filter bar section renders pills from `collection.filters`. JS intercepts click events, updates URL params, and fetches the new collection page section HTML to swap in without full reload.

**When to use:** `ll-collection-filter-bar.liquid` + `ll-filter-bar.js`.

**Trade-offs:** Requires Shopify storefront filters to be enabled on the collection in the Admin (default for all Online Store plans). The fetch-and-swap (section rendering API) approach means the URL updates properly (shareable, back-button safe) without a full reload.

**Key Liquid in `ll-collection-filter-bar.liquid`:**
```liquid
{%- for filter in collection.filters -%}
  {%- if filter.type == 'list' -%}
    {%- for value in filter.values -%}
      <button
        class="ll-pill{% if value.active %} is-active{% endif %}"
        data-ll-filter-url="{{ value.url_to_add }}"
        data-ll-filter-remove="{{ value.url_to_remove }}"
        aria-pressed="{{ value.active }}"
      >
        {{ value.label }} ({{ value.count }})
      </button>
    {%- endfor -%}
  {%- endif -%}
{%- endfor -%}
```

---

## Data Flow

### Add to Cart Flow

```
User clicks "Add to cart" button
    ↓
ll-variant-selector.js reads selected variant ID from [data-ll-selected-variant]
    ↓
POST /cart/add.js  { id: variantId, quantity: 1 }
    ↓
Shopify returns updated cart JSON
    ↓
document fires CustomEvent('ll:cart:open')
    ↓
LLCartDrawer.open() is called
    ↓
GET /cart.js  → render drawer items + bulk progress bar
    ↓
User sees drawer slide in with updated cart
```

### Collection Filter Flow

```
User clicks a pill filter
    ↓
ll-filter-bar.js intercepts click, reads data-ll-filter-url
    ↓
history.pushState(newUrl)  — URL updates, back-button works
    ↓
fetch(newUrl + '&sections=ll-main-collection-grid')
    ↓
Shopify Section Rendering API returns JSON with section HTML
    ↓
document.querySelector('[data-ll-collection-grid]').innerHTML = sectionHtml
    ↓
Re-initialize filter bar event listeners on new DOM
```

### Theme Settings → CSS Tokens Flow

```
Merchant changes color in Theme Editor
    ↓
settings_data.json updated by Shopify
    ↓
layout/theme.liquid re-renders on page load
    ↓
<style>:root { --ll-color-ink: {{ settings.color_ink }} }</style>
    ↓
All sections inherit updated CSS custom properties
    ↓
No SCSS recompile needed — plain CSS vars cascade everywhere
```

---

## Integration Points

### New vs. Modified Files (Explicit)

**Dawn scaffold files to modify:**

| Dawn File | Modification |
|-----------|-------------|
| `layout/theme.liquid` | Add `{%- render 'll-cart-drawer' -%}`, CSS token `<style>` block, `ll-theme.css` link, `ll-global.js` + `ll-cart-drawer.js` script tags |
| `config/settings_schema.json` | Replace entirely with Loris Lots schema (brand colors, fonts, section padding controls) |

**All other files are created new.** The Debut sections, snippets, and templates are not carried forward into the Dawn build. Dawn's own built-in sections (e.g., `header-group.liquid`, `footer-group.liquid`) are either replaced by `ll-*` equivalents or removed from the scaffold entirely.

---

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|--------------|-------|
| ATC button → Cart drawer | `CustomEvent('ll:cart:open')` on `document` | Decoupled; any page can trigger drawer |
| Filter pill → Collection grid | Section Rendering API fetch + innerHTML swap | URL stays in sync via `history.pushState` |
| Theme settings → Section CSS | CSS custom properties via `:root` in `layout/theme.liquid` | No JS needed for color/font changes |
| Variant pill → ATC button price | DOM update within `ll-variant-selector.js` | Scoped to `[data-ll-product-form]` container |
| Bulk progress bar → Cart total | `ll:cart:updated` CustomEvent carries cart JSON | `ll-bulk-progress.js` (small module) listens and recalculates |

---

### External Services

| Service | Integration Pattern | Notes |
|---------|-------------------|-------|
| Shopify Cart API (`/cart/add.js`, `/cart.js`, `/cart/change.js`) | AJAX fetch, JSON response | No SDK needed; plain `fetch()` |
| Shopify Section Rendering API (`?sections=section-id`) | fetch + JSON response containing rendered HTML | Used for filter bar live reload and cart drawer refresh |
| Shopify Storefront Filters | `collection.filters` Liquid object + URL params | Must be enabled in Admin > Collections > Filters |
| Shopify Predictive Search (`/search/suggest.json`) | Deferred — not in v1.0 scope | Placeholder in header schema only |

---

## Anti-Patterns

### Anti-Pattern 1: Carrying Debut Sections into Dawn

**What people do:** Copy `sections/collection-template.liquid` (Debut) into the Dawn scaffold and attempt to adapt it.

**Why it's wrong:** Debut uses `{% include %}` (deprecated), monolithic per-page section files, no OS2.0 block schema, and class conventions (`grid--view-items`, `page-width`) that conflict with Dawn's CSS. Hybrid codebases fail Theme Store review and break the theme editor.

**Do this instead:** Write `ll-main-collection-grid.liquid` from scratch using Dawn's `{% render %}` syntax, OS2.0 section schema with blocks, and `--ll-*` CSS custom property tokens.

---

### Anti-Pattern 2: Putting the Cart Drawer in `templates/cart.json`

**What people do:** Add the cart drawer section only to the cart template.

**Why it's wrong:** The drawer must be accessible from any page (PDP, collection, home). If it lives only in the cart template, the user must navigate to `/cart` to see it, defeating the purpose of a drawer.

**Do this instead:** Include `{%- render 'll-cart-drawer' -%}` in `layout/theme.liquid`, above the `{{ content_for_layout }}` call. The drawer is always present, always hidden, open/closeable from any page.

---

### Anti-Pattern 3: Inline `<style>` per Section with Hard-coded Colors

**What people do:** Write `background: #15131e` directly in section HTML or scoped `<style>` tags.

**Why it's wrong:** The merchant cannot change brand colors from the theme editor. Theme Store review flags hard-coded colors as a customization failure.

**Do this instead:** Define `--ll-color-ink: #15131e` (and variants) in `config/settings_schema.json` as `color` type settings with defaults. In `layout/theme.liquid`, emit `<style>:root { --ll-color-ink: {{ settings.color_ink }} }</style>`. All section CSS references `var(--ll-color-ink)`.

---

### Anti-Pattern 4: One Monolithic JS File

**What people do:** Ship everything in a single `theme.js` (the Debut pattern).

**Why it's wrong:** Every page loads all JS even when most is irrelevant. Dawn's approach is one file per section or concern, included only where needed. Theme Store performance review penalizes unused JS.

**Do this instead:** Include `ll-filter-bar.js` only in `ll-collection-filter-bar.liquid` via `{{ 'll-filter-bar.js' | asset_url | script_tag }}`. Include `ll-product-gallery.js` only in `ll-main-product.liquid`. Only `ll-cart-drawer.js` and `ll-global.js` belong in `layout/theme.liquid` (always needed).

---

## Build Order (Phase Dependencies)

The order below reflects real implementation dependencies — each layer requires the previous.

```
1. SCAFFOLD
   Clone Dawn → prune built-in sections not needed → configure git

2. DESIGN TOKENS (must precede all CSS)
   config/settings_schema.json (color, font settings)
   assets/ll-theme.css (CSS custom property definitions)
   layout/theme.liquid (token injection <style> block + global JS/CSS links)

3. SHARED SNIPPETS (must precede sections that use them)
   snippets/ll-product-card.liquid
   snippets/ll-price.liquid
   snippets/ll-format-badge.liquid
   snippets/ll-pill.liquid
   snippets/ll-icon-*.liquid

4. CART DRAWER (must precede ATC buttons on any page)
   assets/ll-cart-drawer.js    ← AJAX + open/close
   sections/ll-cart-drawer.liquid
   layout/theme.liquid update: render cart-drawer + include cart-drawer.js

5. GLOBAL LAYOUT SECTIONS (must precede page-specific sections)
   sections/ll-announcement-bar.liquid
   sections/ll-header.liquid (cart icon triggers ll:cart:open event)
   sections/ll-footer.liquid

6. HOME PAGE (Home B — Editorial)
   sections/ll-home-hero.liquid
   sections/ll-format-tabs.liquid
   sections/ll-product-carousel.liquid  (reused on PDP as related-products)
   templates/index.json

7. COLLECTION PAGE (Collection A — Pill Filters)
   assets/ll-filter-bar.js
   sections/ll-collection-banner.liquid
   sections/ll-collection-filter-bar.liquid
   sections/ll-main-collection-grid.liquid
   templates/collection.json

8. PRODUCT DETAIL PAGE (PDP A — Classic Gallery)
   assets/ll-product-gallery.js
   assets/ll-variant-selector.js
   sections/ll-main-product.liquid
   sections/ll-related-products.liquid  (reuses ll-product-carousel pattern)
   templates/product.json

9. CART TEMPLATE (minimal — drawer is already global)
   templates/cart.json  (fallback for no-JS; references simple cart items section)

10. THEME STORE COMPLIANCE PASS
    Accessibility audit (focus management, aria-* on drawer, skip links)
    Performance audit (defer/async on all JS, image lazy loading)
    Theme editor QA (all settings round-trip correctly)
```

---

## Scaling Considerations

This is a Shopify-hosted theme — infrastructure scaling is Shopify's responsibility. Theme-side considerations:

| Scale | Architecture Adjustment |
|-------|------------------------|
| 0–10k products | Current architecture; Shopify handles catalog pagination natively |
| 10k+ products | Pill filter approach may show empty states; ensure Shopify's storefront filter counts are used so merchants see `(0)` on exhausted filters |
| Theme Store distribution (many merchants) | Settings defaults must be visually complete without any merchant configuration — hero, format tabs, and product carousel must have meaningful presets |

---

## Sources

- Shopify OS2.0 architecture: official Shopify docs (Sections Everywhere, JSON templates, Section Rendering API) — HIGH confidence
- Dawn theme conventions: direct inspection of Dawn GitHub repository and Theme Store review requirements — HIGH confidence
- Cart drawer pattern: standard Dawn implementation in `assets/cart-drawer.js` and `sections/cart-drawer.liquid` in Dawn source — HIGH confidence
- Storefront Filters API (`collection.filters`, `filter.values`, URL params): Shopify Liquid documentation — HIGH confidence
- Wireframe section mapping: direct inspection of `/wireframes/*.jsx` and screenshots — HIGH confidence
- Debut codebase analysis: direct read of existing theme files — HIGH confidence

---

*Architecture research for: Loris Lots — Dawn OS2.0 Shopify theme*
*Researched: 2026-05-06*
