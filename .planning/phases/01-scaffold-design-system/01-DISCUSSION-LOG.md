# Phase 1: Scaffold & Design System - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-05-06
**Phase:** 01-scaffold-design-system
**Areas discussed:** Starting point, CSS tokens, JS pattern, Locale scope

---

## Starting Point

| Option | Description | Selected |
|--------|-------------|----------|
| Fresh Dawn, Debut as ref | Download latest Dawn via CLI; keep Debut export read-only as reference for section logic | ✓ |
| Fresh Dawn, ignore Debut | Start from Dawn, throw away Debut entirely | |
| Gut Debut in-place | Remove Debut artifacts file-by-file, overlay Dawn structure | |

**User's choice:** Fresh Dawn via `shopify theme init`, Debut export kept as read-only reference.

| Option | Description | Selected |
|--------|-------------|----------|
| Shopify CLI scaffold | `shopify theme init` pulls latest Dawn release | ✓ |
| GitHub release zip | Download specific Dawn release from github.com/Shopify/dawn | |
| You decide | Leave acquisition method to planner | |

**User's choice:** `shopify theme init` (Shopify CLI scaffold).

---

## CSS Tokens

| Option | Description | Selected |
|--------|-------------|----------|
| Primitives only | One `--ll-color-*` per LL token; components reference primitives directly | ✓ |
| Primitives + semantic layer | Add --ll-color-primary, --ll-color-text-default alias layer on top | |

**User's choice:** Primitives-only token layer.

| Option | Description | Selected |
|--------|-------------|----------|
| Inline style block in theme.liquid | `<style>` block in layout/theme.liquid head outputs `:root { --ll-* }` | ✓ |
| Dedicated CSS snippet | `snippets/ll-tokens.liquid` rendered in head | |
| You decide | Leave injection mechanism to planner | |

**User's choice:** Inline `<style>` block in `layout/theme.liquid`.

| Option | Description | Selected |
|--------|-------------|----------|
| Purple default — match wireframes | Ship #5b3df2 as color_accent default | |
| Neutral default for Theme Store | Ship neutral accent; merchants set purple via theme editor | ✓ |
| You decide | Planner picks a sensible Theme Store-compatible default | |

**User's choice:** Neutral default accent for Theme Store compatibility; #5b3df2 reserved for Loris Lots via theme editor.

---

## JS Pattern

| Option | Description | Selected |
|--------|-------------|----------|
| CustomElements (Web Components) | Extends HTMLElement, customElements.define, connectedCallback auto-init | ✓ |
| Plain ES6 classes + DOMContentLoaded | Class takes element arg, manual init loop on DOMContentLoaded | |

**User's choice:** CustomElements / Web Components API — matches Dawn's own conventions.

| Option | Description | Selected |
|--------|-------------|----------|
| Dispatch on document | All ll: events fire on `document` | ✓ |
| Dedicated LLBus singleton | `window.LLBus` EventTarget instance | |
| You decide | Planner picks consistent ll: namespace approach | |

**User's choice:** All `ll:` CustomEvents dispatch and listen on `document`.

| Option | Description | Selected |
|--------|-------------|----------|
| Yes — include skeleton template | `assets/ll-component.js` with connectedCallback, event boilerplate, comments | ✓ |
| No — establish by example | Let Phase 2 cart drawer be the canonical example | |

**User's choice:** Ship `assets/ll-component.js` skeleton template in Phase 1.

---

## Locale Scope

| Option | Description | Selected |
|--------|-------------|----------|
| Dawn's locale set as-is | Inherit all languages Dawn ships; zero modifications | ✓ |
| English only | Start with just en.default.json | |
| You decide | Planner figures out locale strategy | |

**User's choice:** Use Dawn's locale set as-is.

| Option | Description | Selected |
|--------|-------------|----------|
| Defer to later phases | Phase 1 inherits Dawn locales untouched; ll_ keys added in Phases 2–5 | ✓ |
| Define ll_ namespace now | Establish ll_ key structure in en.default.json in Phase 1 | |

**User's choice:** Defer all custom `ll_` locale keys to the phases that introduce those features.

---

## Claude's Discretion

- Exact `settings_schema.json` color/font setting IDs (e.g. `color_bg` vs `color_background`)
- Whether `--ll-*` settings get their own schema section group or fold into Dawn's existing color/typography sections

## Deferred Ideas

- Custom `ll_` locale namespace — deferred to Phases 2–5
- Semantic/alias CSS token layer — deferred to future milestone
- #5b3df2 purple as shipped default — will be set by merchant in theme editor
