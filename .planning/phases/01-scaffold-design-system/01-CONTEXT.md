# Phase 1: Scaffold & Design System - Context

**Gathered:** 2026-05-06
**Status:** Ready for planning

<domain>
## Phase Boundary

Deliver a clean Dawn-based Shopify OS2.0 project running locally via `shopify theme dev`: all Debut artifacts purged, brand tokens flowing through CSS custom properties, and a consistent Vanilla JS (Web Components) component pattern established as the foundation every subsequent phase builds on.

</domain>

<decisions>
## Implementation Decisions

### Starting Point

- **D-01:** Use `shopify theme init` to scaffold a fresh Dawn project — this is the working base for all development.
- **D-02:** Retain the existing Debut export as a read-only reference. When porting section logic (header, footer, collection structure), read the Debut files for reference; do not copy-paste them into Dawn. The Debut export is reference material, not source.

### CSS Custom Properties

- **D-03:** Implement a primitives-only token layer. One `--ll-color-*` or `--ll-font-*` custom property per token in the wireframe LL object (~15 color tokens + font stack). No semantic/alias layer in Phase 1. Token names map directly from LL object keys: `LL.bg` → `--ll-color-bg`, `LL.accent` → `--ll-color-accent`, `LL.ink` → `--ll-color-ink`, etc.
- **D-04:** Inject tokens via an inline `<style>` block in `layout/theme.liquid` head. The block outputs `:root { --ll-color-bg: {{ settings.color_bg }}; --ll-color-ink: {{ settings.color_ink }}; … }` so theme editor changes flow through immediately.
- **D-05:** Ship neutral/generic default values in `config/settings_data.json` for Theme Store compatibility. The purple accent (#5b3df2) is Loris Lots' brand color — merchants set it in the theme editor; it must NOT be the hardcoded default.

### JS Component Architecture

- **D-06:** Use the Web Components API (`customElements.define`) for all interactive components. Each component extends `HTMLElement`, registers as `ll-{component-name}`, and self-inits via `connectedCallback`. This matches Dawn's own conventions.
- **D-07:** All `ll:` CustomEvents dispatch on and listen to `document`. No singleton event bus object. Pattern: `document.dispatchEvent(new CustomEvent('ll:cart:open', { bubbles: true }))` and `document.addEventListener('ll:cart:open', handler)`.
- **D-08:** Phase 1 ships `assets/ll-component.js` — a documented skeleton template with `connectedCallback`, `disconnectedCallback`, event dispatch/listen boilerplate, and `data-ll-*` attribute wiring. Every subsequent phase copies this file as the starting point for new components.

### Locale / i18n

- **D-09:** Inherit Dawn's locale set as-is — all languages Dawn ships out of the box, no modifications. This passes `shopify theme check` without extra work.
- **D-10:** Custom `ll_` locale keys (bulk discount messages, format badge labels, etc.) are deferred entirely to the phases that introduce those features (Phases 2–5). Phase 1 does not define any `ll_` keys.

### Claude's Discretion

- Exact mapping of `settings_schema.json` color/font setting IDs (e.g. `color_bg`, `color_ink`) — planner decides naming as long as they correspond 1:1 to the LL token set.
- Whether to add a `settings_schema.json` section group for "Brand Tokens" or fold the `--ll-*` settings into existing Dawn color/typography sections — planner decides.

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Design Tokens
- `wireframes/primitives.jsx` — Defines the complete LL token object (all ~15 color values + font stack). This is the authoritative source for `--ll-color-*` and `--ll-font-*` custom property values and names.

### Requirements
- `.planning/REQUIREMENTS.md` §Scaffold & Design System (SCAFFOLD-01 through SCAFFOLD-04) — Acceptance criteria for this phase.
- `.planning/ROADMAP.md` §Phase 1 — Success criteria (4 items) that must be TRUE for the phase to be complete.

### Existing Codebase (Read-only Reference)
- `config/settings_schema.json` — Current Debut schema structure. Use as reference for what settings exist; replace theme_info block entirely.
- `assets/theme.scss.liquid` — Current SCSS variable / `{{ settings.* }}` pattern. Replace with CSS custom properties; this file shows what the old pattern looked like.

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- None — Phase 1 starts fresh from Dawn. The Debut export is reference only.

### Established Patterns (from Debut — read, don't copy)
- `assets/theme.scss.liquid`: SCSS `$color-*` vars populated from `{{ settings.color_* }}` — this is the pattern being replaced by CSS custom properties.
- `config/settings_schema.json`: Multi-language setting names using locale objects — Dawn simplifies this; planner should check Dawn's schema structure instead.

### Debut Artifacts to Purge
- 145 `{% include %}` tags → replace with `{% render %}`
- 15 lazysizes.js references → remove; use native `loading="lazy"` and `fetchpriority`
- 42 hardcoded hex colors in `.liquid` files → replace with `var(--ll-color-*)`
- 14 `console.log` calls in JS files → remove
- All `.liquid` templates in `templates/` → replace with JSON templates (OS2.0)
- `theme_info` in `settings_schema.json` → update to Loris Lots metadata

### Integration Points
- `layout/theme.liquid` — receives the `<style>` block injecting `--ll-*` tokens; also includes the component skeleton and CustomEvent bus setup.
- `config/settings_schema.json` — receives new `theme_info` metadata and `--ll-*` color/font settings.

</code_context>

<specifics>
## Specific Ideas

- The `assets/ll-component.js` skeleton should include commented-out examples of `ll:cart:open` dispatch and `ll:cart:updated` listen so Phase 2 developers have copy-paste starting points.
- Token naming convention follows the LL object keys directly: `LL.accent` → `--ll-color-accent`, `LL.ink` → `--ll-color-ink`, `LL.surface` → `--ll-color-surface`, `LL.pill` → `--ll-color-pill`, `LL.pillText` → `--ll-color-pill-text`, `LL.font` → `--ll-font-body`.

</specifics>

<deferred>
## Deferred Ideas

- Custom `ll_` locale key namespace — deferred to Phases 2–5 (each phase adds its own keys)
- Semantic/alias CSS token layer (--ll-color-primary, --ll-color-text-default) — deferred to a future milestone if needed
- Loris Lots purple (#5b3df2) as default theme color — will be set via theme editor, not shipped as default

</deferred>

---

*Phase: 01-scaffold-design-system*
*Context gathered: 2026-05-06*
