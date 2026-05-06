# Stack Research

**Domain:** Shopify Online Store 2.0 custom theme (Dawn base, Theme Store submission)
**Researched:** 2026-05-06
**Confidence:** MEDIUM — Core CLI/Dawn architecture verified against training data through August 2025. Version numbers (CLI, Dawn) flagged where exact current version cannot be confirmed without live web access. Pattern guidance is HIGH confidence from official Shopify developer documentation known at cutoff.

---

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Shopify CLI | 3.x (current: ~3.67+) | Local dev server, theme push/pull/deploy, theme check integration | CLI v3 is the only supported version; v1/v2 were sunset. Installs via npm. Required for `shopify theme dev` hot-reload loop and `shopify theme push` for store deployment |
| Dawn (reference theme) | 14.x (verify at github.com/Shopify/dawn) | OS2.0 scaffold — JSON templates, sections everywhere, correct file structure | Shopify's canonical reference for OS2.0; Theme Store reviewers expect this structure. Starting from Dawn guarantees JSON template structure, correct `{% sections %}` wiring, and accessibility baseline already passing |
| Shopify Liquid | Current (server-rendered) | Templating language — all page rendering | Theme Store requires Liquid; no headless. Server-rendering means zero client-side framework overhead — critical for Core Web Vitals |
| CSS / SCSS (server-compiled) | N/A | Styling | Shopify's CDN compiles `.css.liquid` and `.scss.liquid` files server-side. No local SCSS compile step needed for theme-distributed styles. Write SCSS in `assets/*.scss.liquid` and Shopify handles compilation |
| Vanilla JS (ES6+) | ES2020 target | Cart drawer, filter pills, sticky ATC, section events | Theme Store prohibits jQuery as a required dependency; Dawn itself uses native Web Components and vanilla ES modules. No framework runtime bundle means faster LCP |
| theme-check (via CLI) | Bundled in Shopify CLI 3 | Liquid linting, OS2.0 structure validation, accessibility checks | Shopify CLI 3 bundles theme-check; run `shopify theme check`. Theme Store review runs this automatically — zero violations required for submission |

### Supporting Libraries / APIs

| Library / API | Source | Purpose | When to Use |
|---------------|--------|---------|-------------|
| Shopify Ajax API (`/cart.js`, `/cart/add.js`, `/cart/update.js`) | Shopify CDN (built-in) | Cart drawer: add-to-cart, quantity updates, cart state without page reload | Use for all cart drawer interactions. No npm install — endpoints are available on every Shopify store by default |
| Shopify Section Events (`shopify:section:load` etc.) | Browser custom events (Shopify-injected) | Re-initialize JS when theme editor adds/removes sections dynamically | Must handle in every section's JS; Theme Store requires editor compatibility. Listen on `document` |
| Shopify Predictive Search API (`/search/suggest.json`) | Shopify CDN (built-in) | Search-as-you-type suggestions | If search bar is implemented; no install needed |
| Shopify Customer Privacy API | Shopify CDN (built-in) | GDPR consent banner (required for Theme Store) | Must implement; Theme Store reviewers check for privacy API usage |
| Web Components (native) | Browser (no install) | Cart drawer, modal overlays, variant selectors as custom elements | Dawn uses this pattern throughout (`cart-drawer.js` exports `class CartDrawer extends HTMLElement`). Match this pattern for custom components |
| IntersectionObserver (native) | Browser (no install) | Sticky ATC bar trigger (show/hide based on ATC button visibility) | Use instead of scroll event listeners; better performance, no polling |
| CSS Custom Properties (variables) | Browser (no install) | Theme editor color/font settings propagated to CSS | Dawn uses `--color-background`, `--color-foreground` etc. fed by `settings_schema.json` values via Liquid into `:root {}` in a `base.css` |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| Shopify CLI 3 | Dev server, push, pull, check, packaging | Install via `npm install -g @shopify/cli @shopify/theme`. Requires Node.js 18+. Do NOT use legacy `shopify-themekit` (deprecated) |
| Node.js 18+ | CLI runtime only | Node is only needed to run Shopify CLI — it is NOT a build tool for theme JS/CSS. The theme itself ships vanilla JS files |
| GitHub (Dawn fork) | Version control + Dawn upstream | Fork github.com/Shopify/dawn. Work on feature branches. Pull Dawn upstream changes periodically. Theme Store does not require a specific VCS setup but fork gives clean diff baseline |
| theme-check | Liquid linting + OS2.0 validation | Bundled in CLI 3; run `shopify theme check` from theme root. Separate VS Code extension also available: Shopify Liquid (extension ID: `Shopify.theme-check-vscode`) |
| Shopify Theme Check VS Code Extension | Real-time Liquid linting in editor | Extension ID: `Shopify.theme-check-vscode`. Reads `.theme-check.yml` config from project root. Set `enabled_checks` to match what Theme Store review runs |
| Shopify Partner Dashboard | Theme submission portal | themes.shopify.com/partners — where you upload the `.zip` and submit for review. Separate from any store deployment |

---

## CLI Commands Reference

### Initial Setup

```bash
# Install Shopify CLI (requires Node 18+)
npm install -g @shopify/cli @shopify/theme

# Verify install
shopify version

# Authenticate with Partner account
shopify auth login
```

### Dawn Fork / Project Init

```bash
# Option A: Clone Dawn directly (recommended for full control)
git clone https://github.com/Shopify/dawn.git loris-lots-theme
cd loris-lots-theme
git remote rename origin dawn-upstream
git remote add origin <your-github-repo-url>
git push -u origin main

# Option B: Use CLI to init from Dawn (wraps the above)
shopify theme init loris-lots-theme
# Prompts you to select Dawn — creates a local copy
```

Dawn's file structure after clone:
```
assets/           # JS, CSS, images — no subdirectories (Shopify flat requirement)
config/           # settings_schema.json, settings_data.json
layout/           # theme.liquid (master layout), password.liquid
locales/          # en.default.json + translations (33 required languages for Theme Store)
sections/         # .liquid section files with {% schema %} blocks
snippets/         # reusable .liquid partials
templates/        # JSON templates (OS2.0) — e.g. index.json, product.json
```

### Development Loop

```bash
# Start local dev server (hot-reload against a dev store)
shopify theme dev --store=your-store.myshopify.com

# Push theme to a store (unpublished)
shopify theme push --store=your-store.myshopify.com --unpublished

# Pull live theme changes back to local
shopify theme pull --store=your-store.myshopify.com

# Run theme check (linting + OS2.0 validation)
shopify theme check

# Package theme for Theme Store submission
shopify theme package
# Outputs: loris-lots-theme-<version>.zip
```

### Theme Check Config

Create `.theme-check.yml` in theme root:

```yaml
# .theme-check.yml
TemplateLength:
  enabled: true
  max_length: 200
UnusedAssign:
  enabled: true
LiquidTag:
  enabled: true
```

Run with: `shopify theme check --output=text`

---

## SCSS Compilation Approach

**Verdict: No local build step. Shopify compiles SCSS server-side.**

Dawn has moved away from `.scss.liquid` toward plain `.css` files with CSS Custom Properties. The Debut codebase (current starting point) used `theme.scss.liquid` compiled by Shopify's server. Dawn uses a different approach:

| Approach | Debut (current) | Dawn (target) |
|----------|-----------------|---------------|
| Style file | `assets/theme.scss.liquid` | `assets/base.css` + `assets/component-*.css` |
| Compilation | Shopify server compiles SCSS | Plain CSS — no compilation needed |
| Variables | Liquid interpolation in `.scss.liquid` | CSS Custom Properties set via `{% style %}` tags in `theme.liquid` |
| Import strategy | Single bundled file | Component-level CSS files, each loaded by their section |

**Recommended approach for this project:**

Use Dawn's CSS Custom Properties pattern. In `layout/theme.liquid`, emit `:root { --color-base-accent: {{ settings.color_accent }}; }` from a `{% style %}` block. Reference variables in component CSS files. This eliminates any SCSS dependency and means styles work everywhere with zero tooling.

If SCSS is genuinely needed for a specific complex component: write it as `.scss.liquid`, Shopify will compile it. But the preference is plain CSS.

---

## Vanilla JS Patterns for Interactive UI

### Pattern 1: Cart Drawer (Right Slide-In)

**Architecture: Web Component + Ajax API**

```js
// assets/cart-drawer.js
class CartDrawer extends HTMLElement {
  constructor() {
    super();
    this.addEventListener('click', this.onClose.bind(this));
  }

  open() {
    this.setAttribute('open', '');
    document.body.classList.add('overflow-hidden');
    this.querySelector('[tabindex="-1"]')?.focus();
  }

  close() {
    this.removeAttribute('open');
    document.body.classList.remove('overflow-hidden');
    // Return focus to trigger element
  }

  // Fetch updated cart HTML from Shopify
  async renderContents(parsedState) {
    const response = await fetch(`${routes.cart_url}?section_id=cart-drawer`);
    const text = await response.text();
    const html = new DOMParser().parseFromString(text, 'text/html');
    this.querySelector('.cart-drawer__inner').innerHTML =
      html.querySelector('.cart-drawer__inner').innerHTML;
    this.open();
  }
}
customElements.define('cart-drawer', CartDrawer);
```

**Add-to-cart trigger wires into the form submit:**

```js
// Listen on product form submit, intercept, call /cart/add.js
document.querySelector('form[action="/cart/add"]')
  .addEventListener('submit', async (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    await fetch('/cart/add.js', { method: 'POST', body: formData });
    document.querySelector('cart-drawer').renderContents();
  });
```

**Key requirement:** The cart drawer section (`sections/cart-drawer.liquid`) must have a `{% schema %}` block so it can be rendered via section rendering API (`?section_id=cart-drawer`).

### Pattern 2: Pill Filter Bar (Collection Page)

**Architecture: URL parameter manipulation + fetch, no page reload**

```js
// assets/facets.js
class FacetFiltersForm extends HTMLElement {
  constructor() {
    super();
    this.addEventListener('change', this.onActiveFilterChange.bind(this));
  }

  onActiveFilterChange(event) {
    const form = this.querySelector('form');
    const params = new URLSearchParams(new FormData(form));
    this.renderPage(params.toString());
  }

  async renderPage(queryString) {
    const url = `${window.location.pathname}?${queryString}&section_id=main-collection-product-grid`;
    const response = await fetch(url);
    const text = await response.text();
    const html = new DOMParser().parseFromString(text, 'text/html');
    // Swap grid contents
    document.querySelector('#ProductGridContainer').innerHTML =
      html.querySelector('#ProductGridContainer').innerHTML;
    // Update URL without reload
    history.pushState({}, '', `${window.location.pathname}?${queryString}`);
  }
}
customElements.define('facet-filters-form', FacetFiltersForm);
```

Pill buttons are `<input type="checkbox">` styled as pills. Checking/unchecking fires `change` event, triggering `renderPage`. Dawn's `main-collection-product-grid` section is designed to be fetched this way via `section_id`.

### Pattern 3: Sticky Add-to-Cart Bar (PDP)

**Architecture: IntersectionObserver watching the inline ATC button**

```js
// assets/sticky-atc.js
class StickyAtcBar extends HTMLElement {
  connectedCallback() {
    const inlineAtc = document.querySelector('[data-sticky-atc-trigger]');
    if (!inlineAtc) return;

    const observer = new IntersectionObserver(([entry]) => {
      this.classList.toggle('sticky-atc--visible', !entry.isIntersecting);
    }, { threshold: 0 });

    observer.observe(inlineAtc);
  }
}
customElements.define('sticky-atc-bar', StickyAtcBar);
```

CSS handles the slide-in animation via `transform: translateY(100%)` toggled by `.sticky-atc--visible`. The bar is in `layout/theme.liquid` (or a `{% section %}` call) so it persists across PDP sections.

### Pattern 4: Format Tabs / Carousel (Home Page)

**Architecture: vanilla tab switching with scroll snap for carousel**

```js
// assets/product-carousel.js
class ProductCarousel extends HTMLElement {
  connectedCallback() {
    this.tabButtons = this.querySelectorAll('[data-tab]');
    this.panels = this.querySelectorAll('[data-panel]');
    this.tabButtons.forEach(btn =>
      btn.addEventListener('click', () => this.switchTab(btn.dataset.tab))
    );
  }

  switchTab(target) {
    this.tabButtons.forEach(b => b.setAttribute('aria-selected', b.dataset.tab === target));
    this.panels.forEach(p => p.hidden = p.dataset.panel !== target);
  }
}
customElements.define('product-carousel', ProductCarousel);
```

Carousel scroll: CSS `scroll-snap-type: x mandatory` on the track + `scroll-snap-align: start` on items. No library needed. Prev/next buttons call `scrollBy({ left: ±itemWidth, behavior: 'smooth' })`.

### Section Events (required for all components)

Every JS web component must re-initialize on Theme Editor events:

```js
// Pattern used in every component
document.addEventListener('shopify:section:load', (e) => {
  if (e.target.querySelector('cart-drawer')) {
    // re-init if needed; Web Components auto-fire connectedCallback
  }
});
```

Web Components auto-call `connectedCallback()` when injected by the editor, so this is mostly handled. The main case is cleanup in `shopify:section:unload`.

---

## Theme Store Required Files and Config

These files must be present and correctly structured for Theme Store submission:

| File | Required | Notes |
|------|----------|-------|
| `config/settings_schema.json` | YES | Defines all theme editor settings. Must have `theme_info` object with `theme_name`, `theme_author`, `theme_version`, `theme_documentation_url`, `theme_support_url` |
| `config/settings_data.json` | YES | Default settings values. Committed with sensible defaults |
| `layout/theme.liquid` | YES | Master layout. Must include `{{ content_for_header }}` and `{{ content_for_layout }}` |
| `layout/password.liquid` | YES | Password page layout. Dawn includes this |
| `templates/*.json` | YES | All templates must be JSON (OS2.0). `.liquid` templates are not accepted for new Theme Store submissions |
| `locales/en.default.json` | YES | English locale strings. All user-facing strings must use `t:` keys |
| `locales/*.json` | STRONGLY RECOMMENDED | Theme Store requires 33 languages minimum for acceptance. Dawn provides all 33 |
| `sections/header.liquid` | YES | Must support app blocks (type: `@app`) in schema |
| `sections/footer.liquid` | YES | Must support app blocks |
| `.theme-check.yml` | RECOMMENDED | Configures which checks run; reviewers run theme-check on submission |
| `CHANGELOG.md` | RECOMMENDED | Theme versioning history |

**Metadata in `settings_schema.json` theme_info block:**

```json
{
  "name": "theme_info",
  "theme_name": "Loris Lots",
  "theme_author": "Henry Gillard",
  "theme_version": "1.0.0",
  "theme_documentation_url": "https://your-docs-url.com",
  "theme_support_url": "https://your-support-url.com"
}
```

**JSON template structure (OS2.0 required):**

```json
// templates/product.json
{
  "sections": {
    "main": {
      "type": "main-product",
      "settings": {}
    }
  },
  "order": ["main"]
}
```

**App blocks in section schema (required for Theme Store):**

```liquid
{% schema %}
{
  "name": "Header",
  "blocks": [{ "type": "@app" }]
}
{% endschema %}
```

Header and footer sections must accept `@app` blocks so merchants can install apps that inject into header/footer without code edits.

---

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Dawn fork | Build from blank (empty theme) | Only if you need total control and want zero Dawn opinions; adds significant scaffold work; Dawn is better for Theme Store since reviewers expect its structure |
| Vanilla JS Web Components | Alpine.js | Alpine is acceptable in non-Theme-Store themes; rejected here because Theme Store requires Liquid-only JS patterns and Alpine adds a CDN dependency |
| CSS Custom Properties + plain CSS | SCSS with local build | SCSS is fine if complexity demands it; avoid if you can since it adds a mental model overhead; Dawn has moved to plain CSS |
| Shopify CLI 3 `shopify theme dev` | Webpack / Parcel / Vite build pipeline | External build pipelines (like Slate did) add complexity; Shopify CLI handles everything needed; Vite setups exist but are non-standard for Theme Store |
| IntersectionObserver for sticky ATC | scroll event listener | Scroll listeners fire constantly, hurting INP; IntersectionObserver fires only on boundary crossing |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| Shopify Themekit (legacy CLI) | Deprecated; no longer maintained; no OS2.0 support; `shopify theme dev` does not work with it | Shopify CLI 3 (`@shopify/cli`) |
| Shopify Slate | Deprecated in 2019; build-pipeline approach incompatible with modern Theme Store | Shopify CLI 3 directly |
| jQuery | Theme Store reviewers flag jQuery as an unnecessary dependency; Dawn does not include it; adds ~90KB | Vanilla JS (`fetch`, `querySelector`, `addEventListener`) |
| Alpine.js | CDN-loaded framework; Theme Store prefers zero external JS runtime dependencies | Vanilla JS Web Components |
| React / Vue / Svelte in theme | Not Liquid; ineligible for Theme Store; Hydrogen is the React path but excludes Theme Store | Liquid + Vanilla JS |
| `@import` in SCSS without build step | Shopify's server-side SCSS compiler does not support `@import` across files | Use CSS Custom Properties; or inline all SCSS in a single file |
| Shopify CLI v1 / v2 commands | Syntax is different; many tutorials reference old `shopify login`, `shopify theme serve` — these do not work in CLI 3 | CLI 3: `shopify auth login`, `shopify theme dev` |

---

## Version Compatibility

| Package | Compatible With | Notes |
|---------|-----------------|-------|
| `@shopify/cli` 3.x | Node.js 18+ | Node 16 is end-of-life; use Node 18 LTS or 20 LTS |
| Dawn 14.x | Shopify CLI 3.x | Dawn releases track Shopify platform; use latest Dawn tag |
| theme-check | Bundled in CLI 3 | No separate install needed; `shopify theme check` invokes it |
| `.theme-check.yml` | theme-check 2.x | Check syntax changed between 1.x and 2.x; CLI 3 uses 2.x |

---

## Confidence Notes

- **Shopify CLI 3 command set** — HIGH confidence. CLI 3 architecture (`shopify theme dev`, `shopify theme push`, `shopify theme check`, `shopify theme package`) is well-documented and stable. Commands verified against official Shopify developer docs structure known at training cutoff.
- **Dawn file structure** — HIGH confidence. Dawn's OS2.0 structure (JSON templates, assets flat directory, sections with schema, `{% sections %}` in layout) is stable and well-documented.
- **Exact CLI version number** — LOW confidence. "3.67+" is approximate from training data; verify with `npm show @shopify/cli version` before pinning.
- **Exact Dawn version number** — LOW confidence. "14.x" is approximate; verify at github.com/Shopify/dawn/releases before starting.
- **Theme Store locale count (33 languages)** — MEDIUM confidence. Shopify has increased locale requirements over time; verify current count at shopify.dev/docs/storefronts/themes/store/requirements before packaging.
- **Web Component patterns** — HIGH confidence. Dawn's source code (publicly available) uses this exact pattern; the cart drawer, quantity selector, and variant picker are all `class Foo extends HTMLElement` implementations.

---

## Sources

- Shopify Dawn GitHub (github.com/Shopify/dawn) — file structure, JS patterns, Web Component approach (HIGH confidence, public repo)
- Shopify Developer Docs (shopify.dev/docs/storefronts/themes/tools/cli) — CLI 3 command reference (MEDIUM confidence, training data through Aug 2025)
- Shopify Developer Docs (shopify.dev/docs/storefronts/themes/store/requirements) — Theme Store submission requirements (MEDIUM confidence, training data)
- Existing Debut codebase (theme_export__loris-lots-com-debut) — confirmed: `.scss.liquid` pattern, jQuery/Lodash usage, legacy `.liquid` templates — exactly what NOT to carry forward (HIGH confidence, direct inspection)

---

*Stack research for: Shopify OS2.0 custom theme — Dawn base, Theme Store submission*
*Researched: 2026-05-06*
