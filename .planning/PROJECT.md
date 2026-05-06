# Loris Lots — Shopify Custom Theme

## What This Is

A custom Shopify Online Store 2.0 theme for Loris Lots, a used physical media retailer (DVDs, Blu-rays, CDs, Vinyl). Built on Shopify's Dawn reference theme using Liquid + Vanilla JS, implementing a Shop App-inspired redesign from wireframes. Designed to be deployed for the Loris Lots client store and packaged for sale on the Shopify Theme Store.

## Core Value

A merchant can install this theme, populate it with their catalog, and have a fast, polished used-media storefront without any custom development — and the theme passes Shopify Theme Store review.

## Current Milestone: v1.0 — Shop App Redesign: Core Pages

**Goal:** Implement the wireframe designs as a Dawn-based Shopify OS2.0 theme, built to Theme Store standards, covering all 4 core page types.

**Target features:**
- Dawn scaffold (replaces Debut base, OS2.0-compliant)
- Home page: Editorial layout — hero deal banner, format tabs, product carousels
- Collection page: Pill filter bar + product grid
- Product detail page: Gallery + format variant selector + sticky add-to-cart
- Cart: Right-side slide-in drawer
- Mobile-responsive across all pages
- Theme editor settings (colors, fonts, section controls)
- Theme Store compliance (performance, accessibility, OS2.0 requirements)

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Dawn base scaffold configured for OS2.0 (JSON templates, sections everywhere)
- [ ] Home: Editorial hero with deal banner, format navigation tabs, product carousel sections
- [ ] Collection: Horizontal pill filter bar with live filtering, responsive product grid
- [ ] PDP: Product image gallery, format/condition variant selector, sticky ATC bar
- [ ] Cart: Slide-in right drawer with line item management and checkout CTA
- [ ] All pages mobile-responsive (375px minimum)
- [ ] Theme editor customization: brand colors, typography, section visibility toggles
- [ ] Theme Store compliance: accessibility, performance, Shopify requirements checklist

### Out of Scope

- Hydrogen / headless storefront — not Theme Store eligible
- Multiple layout variants per page (A/B/C) — one best-fit variant per page selected
- Backend customizations — theme only, no app development
- Search results page — deferred to future milestone
- Blog/article templates — deferred to future milestone
- Checkout customization — requires Shopify Plus; out of scope

## Context

- **Base theme**: Migrating from Debut (legacy) to Dawn (OS2.0 reference theme)
- **Client**: Loris Lots — used physical media store with bulk discount model ("$3.49 DVDs for $2.17 each") and format-based navigation (DVDs, CDs, Blu-Ray, Vinyl)
- **Wireframes**: 15-screen Shop App-inspired wireframe set in `/wireframes/` — JSX source files are the primary design spec (colors, spacing, component structure, interaction logic); screenshots in `/wireframes/screenshots/` are visual reference only
- **Wireframe file map**:
  - `primitives.jsx` → design tokens (maps directly to `--ll-*` CSS custom properties)
  - `home-screens.jsx` → Home B layout, hero, carousels
  - `collection-screens.jsx` → Collection A filter bar, grid
  - `product-screens.jsx` → PDP A gallery, variant selector, info panel
  - `cart-screens.jsx` → Cart A drawer, line items, progress bar
  - `mobile-screens.jsx` → mobile breakpoint layouts for all pages
- **Variant selections**: Home B (Editorial), Collection A (Pill filters), PDP A (Classic gallery), Cart A (Right drawer)
- **Theme Store goal**: Theme packaging, review requirements, and theme settings are first-class concerns, not afterthoughts
- **Stack**: Shopify Liquid, CSS Custom Properties, Vanilla JS — no build-step JS frameworks

## Constraints

- **Framework**: Liquid + Vanilla JS only — Theme Store requires Liquid; no Alpine.js or React
- **Compatibility**: Must work on all Shopify plans (Basic and above)
- **Performance**: Theme Store requires Core Web Vitals compliance; no heavy third-party scripts
- **Accessibility**: WCAG 2.0 AA minimum; Theme Store requirement
- **OS2.0**: JSON templates required; section/block schema for all customizable areas

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Rebuild on Dawn, not extend Debut | Debut is legacy (no OS2.0, not Theme Store eligible); Dawn is Shopify's maintained reference | — Pending |
| Liquid + Vanilla JS only | Theme Store eligibility; zero build-step dependencies; maximum compatibility | — Pending |
| One variant per page type | Scope control for v1.0; Theme Store themes pick a strong opinion over infinite flexibility | — Pending |
| Home B (Editorial) selected | Hero deal banner suits Loris Lots' bulk pricing model; format tabs are prominent | — Pending |
| Collection A (Pill filters) selected | Most versatile layout; works for any store type | — Pending |
| PDP A (Classic gallery) selected | Standard, proven layout; maps cleanly to Shopify product options | — Pending |
| Cart A (Right drawer) selected | Modern UX; doesn't interrupt browsing flow | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd:transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd:complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-05-06 — Milestone v1.0 started*
