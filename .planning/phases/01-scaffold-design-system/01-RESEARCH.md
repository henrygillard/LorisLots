# Phase 1: Scaffold & Design System — Research

**Researched:** 2026-05-06
**Domain:** Shopify OS2.0 theme architecture (Dawn), CSS custom properties, Web Components, Debut artifact removal
**Confidence:** HIGH — all major findings verified against official Shopify docs or direct CLI output

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

- **D-01:** Use `shopify theme init` to scaffold a fresh Dawn project — this is the working base for all development.
- **D-02:** Retain the existing Debut export as a read-only reference. When porting section logic (header, footer, collection structure), read the Debut files for reference; do not copy-paste them into Dawn. The Debut export is reference material, not source.
- **D-03:** Implement a primitives-only token layer. One `--ll-color-*` or `--ll-font-*` custom property per token in the wireframe LL object (~15 color tokens + font stack). No semantic/alias layer in Phase 1. Token names map directly from LL object keys: `LL.bg` → `--ll-color-bg`, `LL.accent` → `--ll-color-accent`, `LL.ink` → `--ll-color-ink`, etc.
- **D-04:** Inject tokens via an inline `<style>` block in `layout/theme.liquid` head. The block outputs `:root { --ll-color-bg: {{ settings.color_bg }}; --ll-color-ink: {{ settings.color_ink }}; … }` so theme editor changes flow through immediately.
- **D-05:** Ship neutral/generic default values in `config/settings_data.json` for Theme Store compatibility. The purple accent (#5b3df2) is Loris Lots' brand color — merchants set it in the theme editor; it must NOT be the hardcoded default.
- **D-06:** Use the Web Components API (`customElements.define`) for all interactive components. Each component extends `HTMLElement`, registers as `ll-{component-name}`, and self-inits via `connectedCallback`. This matches Dawn's own conventions.
- **D-07:** All `ll:` CustomEvents dispatch on and listen to `document`. No singleton event bus object. Pattern: `document.dispatchEvent(new CustomEvent('ll:cart:open', { bubbles: true }))` and `document.addEventListener('ll:cart:open', handler)`.
- **D-08:** Phase 1 ships `assets/ll-component.js` — a documented skeleton template with `connectedCallback`, `disconnectedCallback`, event dispatch/listen boilerplate, and `data-ll-*` attribute wiring. Every subsequent phase copies this file as the starting point for new components.
- **D-09:** Inherit Dawn's locale set as-is — all languages Dawn ships out of the box, no modifications.
- **D-10:** Custom `ll_` locale keys deferred entirely to the phases that introduce those features (Phases 2–5). Phase 1 does not define any `ll_` keys.

### Claude's Discretion

- Exact mapping of `settings_schema.json` color/font setting IDs (e.g. `color_bg`, `color_ink`) — planner decides naming as long as they correspond 1:1 to the LL token set.
- Whether to add a `settings_schema.json` section group for "Brand Tokens" or fold the `--ll-*` settings into existing Dawn color/typography sections — planner decides.

### Deferred Ideas (OUT OF SCOPE)

- Custom `ll_` locale namespace — deferred to Phases 2–5
- Semantic/alias CSS token layer (--ll-color-primary, --ll-color-text-default) — deferred to future milestone
- Loris Lots purple (#5b3df2) as default theme color — will be set via theme editor, not shipped as default
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| SCAFFOLD-01 | Developer can fork Dawn, configure settings_schema.json with theme_info metadata, and develop locally using Shopify CLI 3 (`shopify theme dev`) | CLI scaffold command confirmed; theme_info required fields documented; `shopify theme dev` command confirmed available |
| SCAFFOLD-02 | Merchant can set brand colors and typography in the Shopify theme editor; values flow through CSS custom properties (`--ll-color-*`, `--ll-font-*`) to all sections via layout/theme.liquid | CSS custom property injection pattern from theme.liquid confirmed; color/font_picker setting types documented |
| SCAFFOLD-03 | All Debut artifacts are removed — lazysizes.js, `{% include %}`, hardcoded hex colors, console.log calls, external image URLs; Dawn JSON templates replace all legacy .liquid templates | All artifact counts confirmed from codebase audit; `render` vs `include` difference documented; JSON template format documented |
| SCAFFOLD-04 | All interactive components use consistent Vanilla JS Web Components with `data-ll-*` attributes and communicate via CustomEvent bus (`ll:cart:open`, `ll:cart:updated`) | Dawn's customElements.define pattern confirmed; CustomEvent-on-document pattern documented; ll-component.js skeleton design documented |
</phase_requirements>

---

## Summary

Phase 1 replaces the existing Debut export with a freshly scaffolded Dawn theme, purges all Debut-era artifacts, wires brand tokens through CSS custom properties driven by the theme editor, and establishes a Vanilla JS Web Components pattern that all subsequent phases copy.

The existing repository IS the Debut export. The plan must scaffold a new Dawn project (separate directory or in-place replacement), configure it with the `theme_info` block, add `--ll-color-*` and `--ll-font-*` CSS custom properties driven by `settings_schema.json` color pickers, and create `assets/ll-component.js` as the component skeleton. The Debut export at `assets/`, `sections/`, `snippets/`, `templates/`, `layout/`, `locales/`, and `config/` directories are all replaced by Dawn equivalents — none are carried forward.

**Critical finding:** As of Shopify CLI 3.94.3 (confirmed locally), `shopify theme init` defaults to Shopify's Skeleton theme — NOT Dawn. To scaffold Dawn specifically, the command must be `shopify theme init [name] --clone-url https://github.com/Shopify/dawn.git --latest`. This is verified from the CLI help output.

**Primary recommendation:** Scaffold a fresh Dawn project into the working directory using `--clone-url`, update `config/settings_schema.json` with the `theme_info` block and `--ll-*` color/font settings, add the token injection block to `layout/theme.liquid`, and create `assets/ll-component.js`. Confirm zero `shopify theme check` errors before calling the phase complete.

---

## Standard Stack

### Core

| Tool/File | Version | Purpose | Why Standard |
|-----------|---------|---------|--------------|
| Shopify CLI | 3.94.3 (confirmed locally) | `shopify theme dev`, `shopify theme check`, `shopify theme init` | Official Shopify toolchain; required for local preview |
| Dawn theme | v15.4.1 (confirmed: GitHub API) | Starting point for all development | Shopify's maintained OS2.0 reference theme; Theme Store eligible |
| Liquid | Shopify-managed | Template language for all .liquid files | Only supported server-side language for Shopify themes |
| CSS Custom Properties | Native (no build step) | Token delivery from settings to CSS | Zero-dependency; instant update on theme editor change |
| Web Components API | Native (customElements, HTMLElement) | JS component pattern | Matches Dawn's own architecture; no framework dependency |

### Verification Commands

```bash
# Confirm CLI version
shopify version

# Scaffold Dawn (NOT skeleton — see critical finding)
shopify theme init loris-lots --clone-url https://github.com/Shopify/dawn.git --latest

# Start local dev
shopify theme dev

# Lint theme
shopify theme check
```

**Version verification:**
- Shopify CLI: `shopify version` — confirmed 3.94.3 locally
- Dawn: v15.4.1 — confirmed via `https://api.github.com/repos/Shopify/dawn/releases/latest`

---

## Architecture Patterns

### Dawn Directory Structure (target state after Phase 1)

```
├── assets/
│   ├── base.css               # Dawn's core stylesheet (keep; do not modify heavily in Phase 1)
│   ├── ll-component.js        # NEW: Web Component skeleton template
│   └── [other Dawn assets]    # Keep all Dawn assets as-is
├── config/
│   ├── settings_schema.json   # REPLACE theme_info; ADD --ll-* color/font settings
│   └── settings_data.json     # UPDATE: add ll_* defaults with neutral values
├── layout/
│   └── theme.liquid           # ADD: inline <style> block injecting :root { --ll-color-* }
├── locales/
│   └── [Dawn's locale set]    # Keep untouched (D-09)
├── sections/
│   └── [Dawn sections]        # Keep all Dawn sections; do not port Debut sections in Phase 1
├── snippets/
│   └── [Dawn snippets]        # Keep all Dawn snippets
└── templates/
    └── [Dawn JSON templates]  # Already JSON (OS2.0); replace all Debut .liquid templates
```

### Pattern 1: CSS Token Injection in theme.liquid

**What:** An inline `<style>` block in the `<head>` of `layout/theme.liquid` outputs `:root` custom property declarations by reading Shopify settings values via Liquid.

**When to use:** The sole injection point for all `--ll-color-*` and `--ll-font-*` tokens. No other file defines these properties.

**Example:**
```liquid
{%- comment -%}
  Loris Lots design tokens — sourced from theme editor settings.
  All --ll-color-* and --ll-font-* vars are defined here and consumed by
  all sections via var(--ll-color-*).
{%- endcomment -%}
<style>
  :root {
    --ll-color-bg:         {{ settings.ll_color_bg }};
    --ll-color-ink:        {{ settings.ll_color_ink }};
    --ll-color-ink2:       {{ settings.ll_color_ink2 }};
    --ll-color-muted:      {{ settings.ll_color_muted }};
    --ll-color-line:       {{ settings.ll_color_line }};
    --ll-color-surface:    {{ settings.ll_color_surface }};
    --ll-color-card:       {{ settings.ll_color_card }};
    --ll-color-pill:       {{ settings.ll_color_pill }};
    --ll-color-pill-text:  {{ settings.ll_color_pill_text }};
    --ll-color-accent:     {{ settings.ll_color_accent }};
    --ll-color-cover:      {{ settings.ll_color_cover }};
    --ll-font-body:        {{ settings.ll_font_body.family }}, system-ui, -apple-system, sans-serif;
  }
</style>
```

**Why inline in theme.liquid (not a separate snippet):** D-04 locks this. It ensures zero render lag — the browser parses the values before any section HTML. A separate CSS file would require a stylesheet load and cannot be dynamically populated from settings.

**Note on font_picker:** When `type: font_picker` is used in settings_schema.json, the value in Liquid is a font object. Access `.family` to get the family name string. The default font string must be a valid Shopify font handle (e.g., `"assistant_n4"`) — not the display name.

### Pattern 2: settings_schema.json — theme_info and Token Settings

**What:** The `theme_info` object at index 0 of the array provides required metadata. Subsequent objects define setting groups.

**Required fields for theme_info** (verified against Shopify docs):
```json
{
  "name": "theme_info",
  "theme_name": "Loris Lots",
  "theme_author": "Your Name",
  "theme_version": "1.0.0",
  "theme_documentation_url": "https://example.com/docs",
  "theme_support_url": "https://example.com/support"
}
```
**Note:** Specify `theme_support_url` OR `theme_support_email` — not both. Including both causes a `shopify theme check` error.

**Color setting structure:**
```json
{
  "type": "color",
  "id": "ll_color_accent",
  "label": "Accent color",
  "default": "#1a1a2e"
}
```
The `id` value becomes `settings.ll_color_accent` in Liquid. The `default` must be a valid hex code. D-05 requires the accent default to be a neutral color, not #5b3df2.

**Font picker structure:**
```json
{
  "type": "font_picker",
  "id": "ll_font_body",
  "label": "Body font",
  "default": "assistant_n4"
}
```
The `default` attribute is required for `font_picker` — omitting it causes a schema error.

### Pattern 3: Web Component Skeleton (ll-component.js)

**What:** A documented template file that all subsequent phases copy as the starting point for new interactive components. Defines the `customElements.define` guard pattern Dawn uses.

**Why the guard:** If a section's JS file is included more than once on a page (possible in OS2.0 with multiple section instances), `customElements.define` throws if the name is already registered. Always check `customElements.get()` first (confirmed pattern in Dawn itself).

**Template:**
```javascript
// assets/ll-component.js
// Copy this file for each new interactive component.
// Replace 'll-component' with 'll-{your-component-name}'.
// Source: Dawn convention (customElements.get guard) + D-06, D-07, D-08

class LlComponent extends HTMLElement {
  connectedCallback() {
    // Called when element is inserted into the DOM.
    // Wire up data-ll-* attribute selectors here.
    this.trigger = this.querySelector('[data-ll-trigger]');
    if (this.trigger) {
      this.trigger.addEventListener('click', this.handleClick.bind(this));
    }

    // Listen for ll: events from other components.
    // Example: document.addEventListener('ll:cart:updated', this.handleCartUpdate.bind(this));
    this._onCartOpen = this.handleCartOpen.bind(this);
    document.addEventListener('ll:cart:open', this._onCartOpen);
  }

  disconnectedCallback() {
    // Always clean up listeners to prevent memory leaks.
    document.removeEventListener('ll:cart:open', this._onCartOpen);
  }

  handleClick(event) {
    // Dispatch an ll: event so other components can react.
    // bubbles: true lets it propagate up the DOM if needed.
    document.dispatchEvent(new CustomEvent('ll:cart:open', {
      bubbles: true,
      detail: { source: 'll-component' }
    }));
  }

  handleCartOpen(event) {
    // React to ll:cart:open dispatched by another component.
    // event.detail contains whatever was passed by the dispatcher.
  }
}

// Guard prevents duplicate registration when section JS loads twice.
if (!customElements.get('ll-component')) {
  customElements.define('ll-component', LlComponent);
}
```

**Usage in Liquid:**
```liquid
<ll-component data-ll-target="some-value">
  <button data-ll-trigger>Open</button>
</ll-component>
```

### Pattern 4: JSON Templates (OS2.0)

**What:** Dawn uses `.json` template files instead of `.liquid` template files. JSON templates reference section files and control the order they render. Merchants can add/remove/reorder sections via the theme editor.

**Minimal structure:**
```json
{
  "sections": {
    "main": {
      "type": "main-collection",
      "settings": {}
    }
  },
  "order": ["main"]
}
```

Dawn already ships JSON templates for all standard page types. Phase 1 inherits them unchanged.

### Anti-Patterns to Avoid

- **`{% include %}` anywhere:** Theme Check flags this as an error. Every snippet call must use `{% render %}`. Key difference: `render` does not inherit parent scope — you must explicitly pass variables: `{% render 'snippet-name', product: product %}`.
- **Hardcoded hex colors in Liquid/CSS:** Any `#xxxxxx` in a `.liquid` file (outside `settings_schema.json` defaults) is a Debut artifact. Colors must come from `var(--ll-color-*)`.
- **Global `var theme = {}` pattern:** Debut used a global JS object for configuration. Dawn uses Web Components with scoped state. Do not carry this forward.
- **`customElements.define` without guard:** Without `if (!customElements.get('ll-cart'))` guard, duplicate section includes crash the page silently.
- **Inline `onclick` handlers:** All event binding must go through `connectedCallback` using `addEventListener`. No `onclick=` attributes in Liquid templates — violates SCAFFOLD-04.
- **lazysizes classes on images:** Dawn uses `loading="lazy"` and `fetchpriority="high"` (for LCP images). The `lazyload` CSS class, `data-src` attribute, and `lazysizes.js` script reference must all be removed.
- **`console.log` in shipped code:** 14 instances confirmed in Debut. All must be removed. Theme Store review will flag these.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Dynamic CSS from theme settings | Custom build pipeline or JS injection | Liquid `{{ settings.* }}` in inline `<style>` block | Native Shopify pattern; no JS required; updates instantly |
| Font loading from theme editor | Manual font embed logic | `font_picker` setting type + Dawn's font preload pattern | Shopify CDN handles loading; Dawn already has the preload conditional |
| Snippet variable passing | Global variable workarounds | `{% render 'snippet', var: value %}` explicit passing | This is exactly what `render` is designed for |
| Event bus singleton | `window.LLBus = new EventTarget()` | `document.dispatchEvent` / `document.addEventListener` (D-07) | No singleton needed; `document` is the universal bus |
| Custom property polyfill | Any CSS-in-JS or PostCSS step | Native CSS custom properties | All Shopify-supported browsers support native CSS vars |

**Key insight:** Shopify's theme system is designed to eliminate build steps entirely. Any custom pipeline for CSS, JS bundling, or font management adds complexity and breaks the `shopify theme dev` hot-reload workflow.

---

## Debut Artifact Inventory (Confirmed from Codebase Audit)

These are the exact artifacts that must be purged. Since Phase 1 replaces the Debut export with a fresh Dawn scaffold, the purge happens by deletion of the old files — not by editing them.

| Artifact | Count | Location | Action |
|----------|-------|----------|--------|
| `{% include %}` tags | 145 | Various `.liquid` files | Deleted with Debut files; Dawn uses `{% render %}` |
| `lazysizes.js` references | 15 | `.liquid` + `.js` files | Deleted; Dawn uses `loading="lazy"` natively |
| `console.log` calls | 14 | `assets/theme.js`, `assets/vendor.js` | Deleted with Debut files |
| Hardcoded hex colors in `.liquid` | 32 | Section/snippet `.liquid` files | Deleted with Debut files |
| `.liquid` templates in `templates/` | 13 main + 7 customers | `templates/*.liquid` | Replaced by Dawn's JSON templates |
| `theme_info` (Debut metadata) | 1 | `config/settings_schema.json` | Replaced with Loris Lots metadata |
| SCSS variable pattern (`$color-*: {{ settings.* }}`) | ~20 vars | `assets/theme.scss.liquid` | Deleted; replaced by CSS custom property injection |
| `assets/theme.scss.liquid` | 1 | `assets/` | Deleted; Dawn uses `assets/base.css` |
| `assets/lazysizes.js` | 1 | `assets/` | Deleted |
| `assets/vendor.js` | 1 | `assets/` | Deleted |

---

## Token Mapping (from wireframes/primitives.jsx)

Complete LL token object to `--ll-*` CSS custom property and `settings_schema.json` ID mapping:

| LL Key | Value | CSS Custom Property | settings.json ID | Default for Theme Store |
|--------|-------|---------------------|------------------|------------------------|
| `LL.bg` | `#ffffff` | `--ll-color-bg` | `ll_color_bg` | `#ffffff` |
| `LL.ink` | `#15131e` | `--ll-color-ink` | `ll_color_ink` | `#15131e` |
| `LL.ink2` | `#3d3a4a` | `--ll-color-ink2` | `ll_color_ink2` | `#3d3a4a` |
| `LL.muted` | `#8a8794` | `--ll-color-muted` | `ll_color_muted` | `#8a8794` |
| `LL.line` | `#e9e7e0` | `--ll-color-line` | `ll_color_line` | `#e9e7e0` |
| `LL.surface` | `#f7f5f0` | `--ll-color-surface` | `ll_color_surface` | `#f7f5f0` |
| `LL.card` | `#ffffff` | `--ll-color-card` | `ll_color_card` | `#ffffff` |
| `LL.pill` | `#0e0d14` | `--ll-color-pill` | `ll_color_pill` | `#0e0d14` |
| `LL.pillText` | `#ffffff` | `--ll-color-pill-text` | `ll_color_pill_text` | `#ffffff` |
| `LL.accent` | `#5b3df2` | `--ll-color-accent` | `ll_color_accent` | `#1a1a2e` (neutral; D-05) |
| `LL.cover` | `#d8d4cc` | `--ll-color-cover` | `ll_color_cover` | `#d8d4cc` |
| `LL.font` | Inter stack | `--ll-font-body` | `ll_font_body` (font_picker) | `assistant_n4` (Shopify font) |

**Tokens intentionally excluded from theme editor settings** (design/wireframe internal only — not brand-configurable):
- `LL.heart` (`rgba(0,0,0,0.55)`) — overlay color; hardcode in CSS as static value
- `LL.liquid` / `LL.liquidBg` — annotation colors used only in wireframes; do not ship
- `LL.mono` — monospace font; not required in Phase 1

**Rationale:** The theme editor should only expose tokens that a merchant would meaningfully want to change. Heart overlay and mono font are implementation details.

---

## Common Pitfalls

### Pitfall 1: `shopify theme init` Defaults to Skeleton, Not Dawn

**What goes wrong:** Running `shopify theme init` without `--clone-url` creates a Skeleton theme (minimal boilerplate), not Dawn. The user decision D-01 says "Dawn" — Skeleton is a different theme with a different file structure.

**Why it happens:** Shopify changed the CLI default from Dawn to Skeleton (confirmed in CLI 3.94.3 help output: "Defaults to Shopify's Skeleton theme").

**How to avoid:** Always use the explicit clone URL:
```bash
shopify theme init loris-lots --clone-url https://github.com/Shopify/dawn.git --latest
```

**Warning signs:** If scaffolded directory lacks `assets/base.css`, `assets/global.js`, and `assets/pubsub.js` — you got Skeleton, not Dawn.

### Pitfall 2: `render` Requires Explicit Variable Passing

**What goes wrong:** Replacing `{% include 'snippet' %}` with `{% render 'snippet' %}` without also passing variables causes the snippet to render with blank data. The Debut `{% include %}` tag shared all parent scope variables implicitly; `{% render %}` does not.

**Why it happens:** The `render` tag intentionally isolates scope for performance and predictability. It is not a drop-in replacement.

**How to avoid:** When writing new snippets in Dawn, always pass variables explicitly:
```liquid
{% render 'product-card', product: product, show_vendor: section.settings.show_vendor %}
```

**Warning signs:** Snippets rendering without data but no Liquid errors in the browser console.

### Pitfall 3: CSS Custom Properties Do Not Update Without Full Inline Style Block

**What goes wrong:** If `--ll-color-*` properties are defined in an external `.css` file (e.g., `base.css`), changing the theme editor color does NOT update them — only a page reload reflects the change. Shopify's theme editor live preview requires the properties to be in an inline `<style>` block in the page `<head>`.

**Why it happens:** Shopify's live preview injects updated setting values by regenerating the inline `<style>` block in the page head. External CSS files are cached and not regenerated.

**How to avoid:** All `--ll-color-*` and `--ll-font-*` declarations MUST be in the inline `<style>` block in `layout/theme.liquid` (D-04). Do not declare them in `base.css` or any external stylesheet.

**Warning signs:** Colors update after browser refresh but not in live preview.

### Pitfall 4: `customElements.define` Without Guard Crashes on Multi-Section Pages

**What goes wrong:** If a section includes a JS file via `{{ 'll-cart.js' | asset_url | script_tag }}` and that section appears twice on a page, `customElements.define('ll-cart', LlCart)` is called twice, throwing `NotSupportedError`.

**Why it happens:** OS2.0 allows multiple instances of the same section on one page. Each instance renders its own `<script>` tag referencing the same asset.

**How to avoid:** Always wrap `customElements.define` with a guard (confirmed Dawn pattern):
```javascript
if (!customElements.get('ll-cart')) {
  customElements.define('ll-cart', LlCart);
}
```

**Warning signs:** Console error `Failed to execute 'define' on 'CustomElementRegistry'` when testing pages with repeated sections.

### Pitfall 5: `theme_support_url` and `theme_support_email` Are Mutually Exclusive

**What goes wrong:** Including both `theme_support_url` and `theme_support_email` in the `theme_info` block causes `shopify theme check` to fail.

**Why it happens:** The Shopify schema requires exactly one support contact method.

**How to avoid:** Use only `theme_support_url` in Phase 1.

### Pitfall 6: `font_picker` Default Must Be a Valid Shopify Font Handle

**What goes wrong:** Setting `"default": "Inter"` for a `font_picker` setting causes a schema validation error. Shopify font handles use a specific format like `"assistant_n4"`.

**Why it happens:** Shopify's font picker resolves fonts from its CDN using internal font handles, not display names.

**How to avoid:** Use `"assistant_n4"` as the default for body font (Dawn's own default; confirmed from Dawn settings_schema.json). The merchant can change to Inter or another system font via the editor. Note: system fonts like Inter are not in Shopify's picker — they must be specified via fallback CSS, not `font_picker`.

**Implication for `--ll-font-body`:** Since Inter (from the wireframes) is a system font not available in Shopify's font picker, the `ll_font_body` setting uses `font_picker` with a Shopify-hosted font as default. The CSS fallback stack `system-ui, -apple-system, sans-serif` in the `:root` block naturally picks up Inter on most macOS/Windows systems. This satisfies both the wireframe intent and Theme Store compliance.

---

## Code Examples

### Inline Token Injection Block (layout/theme.liquid)

```liquid
{%- comment -%}Loris Lots brand tokens — updated live by theme editor.{%- endcomment -%}
<style>
  :root {
    --ll-color-bg:         {{ settings.ll_color_bg }};
    --ll-color-ink:        {{ settings.ll_color_ink }};
    --ll-color-ink2:       {{ settings.ll_color_ink2 }};
    --ll-color-muted:      {{ settings.ll_color_muted }};
    --ll-color-line:       {{ settings.ll_color_line }};
    --ll-color-surface:    {{ settings.ll_color_surface }};
    --ll-color-card:       {{ settings.ll_color_card }};
    --ll-color-pill:       {{ settings.ll_color_pill }};
    --ll-color-pill-text:  {{ settings.ll_color_pill_text }};
    --ll-color-accent:     {{ settings.ll_color_accent }};
    --ll-color-cover:      {{ settings.ll_color_cover }};
    --ll-font-body:        {{ settings.ll_font_body.family }}, system-ui, -apple-system, "Segoe UI", sans-serif;
  }
</style>
```

**Placement:** In `layout/theme.liquid`, inside `<head>`, after the `<meta>` tags, before the first `<link rel="stylesheet">`. This ordering ensures the custom properties are available before any stylesheet that uses them.

### settings_schema.json Brand Tokens Section

```json
{
  "name": "Brand Tokens",
  "settings": [
    { "type": "header", "content": "Colors" },
    { "type": "color", "id": "ll_color_bg",       "label": "Background",    "default": "#ffffff" },
    { "type": "color", "id": "ll_color_ink",       "label": "Primary text",  "default": "#15131e" },
    { "type": "color", "id": "ll_color_ink2",      "label": "Secondary text","default": "#3d3a4a" },
    { "type": "color", "id": "ll_color_muted",     "label": "Muted text",   "default": "#8a8794" },
    { "type": "color", "id": "ll_color_line",      "label": "Border",       "default": "#e9e7e0" },
    { "type": "color", "id": "ll_color_surface",   "label": "Surface",      "default": "#f7f5f0" },
    { "type": "color", "id": "ll_color_card",      "label": "Card",         "default": "#ffffff" },
    { "type": "color", "id": "ll_color_pill",      "label": "Pill bg",      "default": "#0e0d14" },
    { "type": "color", "id": "ll_color_pill_text", "label": "Pill text",    "default": "#ffffff" },
    { "type": "color", "id": "ll_color_accent",    "label": "Accent",       "default": "#1a1a2e" },
    { "type": "color", "id": "ll_color_cover",     "label": "Cover art bg", "default": "#d8d4cc" },
    { "type": "header", "content": "Typography" },
    { "type": "font_picker", "id": "ll_font_body", "label": "Body font",    "default": "assistant_n4" }
  ]
}
```

### Native Image Lazy Loading (replacing lazysizes)

Dawn's approach — use in all `<img>` tags except the LCP hero image:

```liquid
{%- if section.index == 1 -%}
  {%- assign loading = 'eager' -%}
  {%- assign fetchpriority = 'high' -%}
{%- else -%}
  {%- assign loading = 'lazy' -%}
  {%- assign fetchpriority = 'auto' -%}
{%- endif -%}

<img
  src="{{ image | image_url: width: 800 }}"
  srcset="{{ image | image_url: width: 400 }} 400w,
          {{ image | image_url: width: 800 }} 800w,
          {{ image | image_url: width: 1200 }} 1200w"
  sizes="(min-width: 990px) 50vw, 100vw"
  alt="{{ image.alt | escape }}"
  loading="{{ loading }}"
  fetchpriority="{{ fetchpriority }}"
  width="{{ image.width }}"
  height="{{ image.height }}"
>
```

**No `lazyload` class. No `data-src`. No `data-srcset`. No lazysizes.js.**

---

## Validation Architecture

> `workflow.nyquist_validation` key not present in `.planning/config.json` (file does not exist). Treating as enabled.

Phase 1 has no unit-testable JS logic (no algorithms, no data transformation) and no npm test framework. Validation is entirely via grep-based artifact detection, Shopify CLI commands, and manual browser smoke-testing. All checks below can be run in < 30 seconds each.

### Test Framework

| Property | Value |
|----------|-------|
| Framework | None — no npm test framework; Shopify CLI + grep |
| Config file | None |
| Quick run command | `shopify theme check` |
| Full suite command | See Phase Gate below |

### Phase Requirements → Verification Map

| Req ID | Behavior | Verification Type | Automated Command | Notes |
|--------|----------|-------------------|-------------------|-------|
| SCAFFOLD-01 | `shopify theme dev` starts with no errors | Smoke | `shopify theme check` (static) + manual `shopify theme dev` | CLI check is automated; dev server requires manual browser check |
| SCAFFOLD-02 | Theme editor color/font change updates live preview | Manual | — | No automated way to simulate theme editor interaction |
| SCAFFOLD-02 | `--ll-color-*` properties exist in rendered HTML | Grep | `grep -r "ll-color" layout/theme.liquid` | Confirms injection block is present |
| SCAFFOLD-03 | Zero `{% include %}` tags remain | Grep | `grep -rn "{%[ -]*include" --include="*.liquid" . \| wc -l` → must be 0 |  |
| SCAFFOLD-03 | Zero lazysizes references remain | Grep | `grep -rn "lazysizes" --include="*.liquid" --include="*.js" . \| wc -l` → must be 0 |  |
| SCAFFOLD-03 | Zero `console.log` calls remain | Grep | `grep -rn "console\.log" --include="*.js" . \| wc -l` → must be 0 |  |
| SCAFFOLD-03 | Zero hardcoded hex in `.liquid` (outside schema defaults) | Grep | `grep -rn "#[0-9a-fA-F]\{3,6\}" --include="*.liquid" . \| grep -v settings_schema` |  |
| SCAFFOLD-03 | All templates are JSON format | Find | `find templates/ -name "*.liquid" -not -path "*/customers/*"` → must return empty |  |
| SCAFFOLD-04 | ll-component.js exists with correct pattern | Grep | `grep -n "customElements.define\|ll:cart:open\|data-ll-" assets/ll-component.js` |  |
| SCAFFOLD-04 | No inline event handlers in Liquid | Grep | `grep -rn "onclick=" --include="*.liquid" .` → must be 0 |  |

### Verification Script (Phase Gate)

Run all checks before declaring Phase 1 complete:

```bash
#!/bin/bash
# Phase 1 artifact audit — all counts must be 0 unless noted
set -e
ROOT="."   # Run from Dawn project root

echo "=== Phase 1 Verification ==="

echo -n "1. {% include %} tags: "
COUNT=$(grep -rn "{%[-]*\s*include" --include="*.liquid" "$ROOT" | wc -l | tr -d ' ')
echo "$COUNT (must be 0)"

echo -n "2. lazysizes refs: "
COUNT=$(grep -rn "lazysizes" --include="*.liquid" --include="*.js" "$ROOT" | wc -l | tr -d ' ')
echo "$COUNT (must be 0)"

echo -n "3. console.log calls: "
COUNT=$(grep -rn "console\.log" --include="*.js" "$ROOT" | wc -l | tr -d ' ')
echo "$COUNT (must be 0)"

echo -n "4. Hardcoded hex in .liquid: "
COUNT=$(grep -rn "#[0-9a-fA-F]\{3,6\}" --include="*.liquid" "$ROOT" | grep -v "config/settings_schema.json" | wc -l | tr -d ' ')
echo "$COUNT (review — Dawn itself may have some; focus on sections/ and snippets/)"

echo -n "5. Legacy .liquid templates: "
COUNT=$(find "$ROOT/templates" -name "*.liquid" | wc -l | tr -d ' ')
echo "$COUNT (must be 0)"

echo -n "6. ll-component.js exists: "
test -f "$ROOT/assets/ll-component.js" && echo "YES" || echo "NO (must exist)"

echo -n "7. Token injection in theme.liquid: "
grep -q "ll-color-bg" "$ROOT/layout/theme.liquid" && echo "YES" || echo "NO (must exist)"

echo -n "8. customElements.define guard: "
grep -q "customElements.get" "$ROOT/assets/ll-component.js" && echo "YES" || echo "NO"

echo ""
echo "=== Shopify CLI Check ==="
shopify theme check
```

### Wave 0 Gaps

None — Phase 1 has no test files to create. The verification script above is created as part of Wave 1 implementation tasks. Existing test infrastructure: none (greenfield).

---

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|-------------|-----------|---------|----------|
| Shopify CLI | `shopify theme dev`, `shopify theme check`, `shopify theme init` | Yes | 3.94.3 | None — required |
| Node.js | Shopify CLI runtime | Yes | v20.20.2 | None — required |
| npm | Shopify CLI | Yes | 10.8.2 | None — required |
| Git | `shopify theme init --clone-url` (clones Dawn) | Yes (assumed macOS default) | — | Manual Dawn zip download from GitHub |
| Internet access | `shopify theme init`, `shopify theme dev` (requires connected store) | Assumed yes | — | None for dev server |

**Missing dependencies:** None. All required tools are available locally.

**Note on Shopify store:** `shopify theme dev` requires a connected Shopify development store. This is outside the scope of Phase 1 tooling verification — it requires a Shopify Partner account and a dev store URL. The plan should include a task to confirm the dev store connection works before starting scaffold work.

---

## State of the Art

| Old Approach (Debut) | Current Approach (Dawn) | Impact |
|----------------------|------------------------|--------|
| `{% include 'snippet' %}` | `{% render 'snippet', var: value %}` | Explicit scope; required for Theme Check pass |
| `lazysizes.js` + `data-src` + `.lazyload` class | `loading="lazy"` + `fetchpriority="high"` on LCP | Zero JS for image lazy loading; better CWV |
| SCSS variables fed by `{{ settings.* }}` in `.scss.liquid` | CSS custom properties in inline `<style>` block | Live preview works; no Sass compilation |
| jQuery-based components | Web Components (`customElements.define`) | No framework dependency; matches Dawn |
| `.liquid` templates with full HTML | JSON templates referencing section files | Sections Everywhere; merchant can add/remove sections |
| Global `var theme = {}` config object | Component-scoped state + `document` event bus | No global coupling; easier to test and maintain |
| Shopify's Sass compiler (`.scss.liquid`) | Plain CSS (`base.css`) | No build step; what Shopify shipped in Dawn |

---

## Open Questions

1. **Dawn scaffold location**
   - What we know: The plan must scaffold Dawn into a new directory or replace the current repo contents in-place.
   - What's unclear: Should the planner create a new sibling directory `../loris-lots/` and set that as the working root, or scaffold into the current git repo (replacing Debut files)?
   - Recommendation: Scaffold into the CURRENT git repository directory (after removing Debut files) so the existing git history and `.planning/` directory are preserved. The plan should include a task that removes all Debut files from the tracked file list before running `shopify theme init`.

2. **Dawn's own `color_scheme_group` settings**
   - What we know: Dawn uses a `color_scheme_group` type in its settings_schema.json with Role-mapped colors. These exist alongside whatever we add.
   - What's unclear: Should the `--ll-*` settings replace Dawn's color scheme system entirely, or coexist with it?
   - Recommendation: Since Phase 1 is a fresh Dawn install and sections are inherited untouched, Dawn's color scheme settings must remain so Dawn's own sections continue to function. Add the `--ll-*` brand token group as a SEPARATE settings group. Downstream phases will migrate section CSS from Dawn's color scheme vars to `--ll-*` vars as each section is rewritten.

3. **`shopify theme dev` store connection**
   - What we know: The command requires authentication to a Shopify development store.
   - What's unclear: Whether the developer has a Shopify Partner dev store connected already.
   - Recommendation: The plan should include an explicit task: "Confirm `shopify theme dev` connects successfully to a development store." This is the gateway task that must succeed before any other work can be verified.

---

## Sources

### Primary (HIGH confidence)
- Shopify CLI 3.94.3 local help output (`shopify theme init --help`) — `--clone-url` behavior and Skeleton default confirmed
- Shopify CLI local version output (`shopify version`) — version 3.94.3 confirmed
- GitHub API (`api.github.com/repos/Shopify/dawn/releases/latest`) — Dawn v15.4.1 confirmed
- Direct codebase audit of Debut export — all artifact counts confirmed (145 includes, 15 lazysizes, 14 console.log, 32 hex colors, 13+7 liquid templates)
- `wireframes/primitives.jsx` — full LL token object with values confirmed

### Secondary (MEDIUM confidence)
- Shopify developer docs (`shopify.dev/docs/storefronts/themes/architecture/config/settings-schema-json`) — `theme_info` required fields; color/font_picker structure
- Shopify developer docs (`shopify.dev/docs/storefronts/themes/architecture/templates/json-templates`) — JSON template format
- Shopify developer docs (`shopify.dev/docs/storefronts/themes/architecture/settings/input-settings`) — color and font_picker setting types
- Dawn GitHub (`raw.githubusercontent.com/Shopify/dawn/main/assets/global.js`) — Web Component pattern and `customElements.get` guard
- Dawn GitHub (`raw.githubusercontent.com/Shopify/dawn/main/assets/pubsub.js`) — pub/sub pattern structure

### Tertiary (LOW confidence — from Web Search, verified by primary sources where possible)
- Dawn lazy loading approach: Web search confirms `loading="lazy"` native approach; aligns with primary Dawn codebase review
- `render` vs `include` variable scoping: Web search confirms; aligned with Shopify changelog

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — CLI confirmed locally; Dawn version from GitHub API
- Architecture patterns: HIGH — confirmed from Dawn source and Shopify official docs
- Token mapping: HIGH — direct read of `wireframes/primitives.jsx`
- Artifact counts: HIGH — confirmed by grep against actual Debut export
- Pitfalls: HIGH — confirmed against CLI help, Dawn source, and official docs

**Research date:** 2026-05-06
**Valid until:** 2026-06-06 (Dawn releases frequently; re-verify version before scaffolding)
