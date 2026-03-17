# ADR-0002: Portfolio Showcase Strategy — Beyond Theme Development

**Date:** 2026-03-17
**Status:** Proposed
**Context:** After building 6 Shopify OS 2.0 themes, identify gaps in the portfolio and plan showcase projects that demonstrate advanced Shopify development skills.

---

## Current Portfolio Coverage

### 6 Themes Built

| # | Theme | Directory | Target | Files | Key Features |
|---|---|---|---|---|---|
| 1 | BrewMaster | `themes/starter-minimal` | Small boutiques | 76 | Clean typography, whitespace-focused |
| 2 | ArtVault | `themes/starter-bold` | Art/creative studios | 69 | Glassmorphism, marquee, editorial layouts |
| 3 | CodeCraft Academy | `themes/starter-developer` | Agencies, SaaS | 57 | Terminal UI, dark mode, comparison table |
| 4 | Patina | `themes/starter-luxury` | Fine jewelry | 52 | Transparent header, hover-to-reveal cards |
| 5 | PawHaven | `themes/starter-pets` | Pet supplies | 52 | Category carousel, mosaic grid, scroll reveals |
| 6 | Zenith | `themes/starter-fitness` | Fitness & wellness | 64 | Quick-view, comparison, countdown, multi-step cart, mega menu, bottom tab bar, dark/light toggle, FAQ search, animated counters |

### Skills Already Demonstrated
- Shopify OS 2.0 architecture (JSON templates, sections, blocks, snippets)
- Schema design with all common setting types
- Section groups (header-group.json, footer-group.json)
- Translation/localization with `{{ 'key' | t }}`
- Responsive, mobile-first CSS without frameworks
- Vanilla JS for interactivity (no jQuery, no frameworks)
- AJAX Cart API (`/cart/add.js`, `/cart.js`, `/cart/change.js`)
- `<dialog>` element for modals (quick-view, compare)
- WCAG accessibility (aria attributes, keyboard nav, screen reader labels)
- Advanced UI: countdown timer, recently viewed (localStorage), multi-step cart, FAQ with search/filter, product comparison, swipeable cards, dark mode toggle, parallax, video backgrounds, animated counters, mega menu, bottom tab bar
- Customer account templates
- CSS custom properties for theming
- `{% doc %}` headers on snippets
- `{% content_for 'blocks' %}` with standalone block files

---

## Identified Gaps

1. **No Shopify App development** — no custom app, no app proxy, no app embed
2. **No Checkout Extensions** — most in-demand Shopify skill currently
3. **No Shopify Functions** — discounts, delivery, payment customization
4. **No Metaobjects** — no custom content modeling
5. **No Storefront API / Headless** — everything is Liquid server-rendered
6. **No Theme App Extensions** — no app blocks injecting into merchant themes
7. **No Webhooks or backend integration**
8. **No performance optimization showcase** — no Lighthouse audit, no documented optimization
9. **No multi-currency / multi-language** — no Markets support
10. **No predictive search** — no Search API or predictive search endpoint
11. **No product filtering** — no Storefront Filtering API usage
12. **No cart drawer** — Zenith has multi-step cart page but no slide-out drawer

---

## Proposed Showcase Projects

### Project 1: Cart Drawer with Upsells (Priority 1)

**What:** Slide-out cart drawer with product recommendations, free shipping progress bar, discount code application, gift wrapping line properties.

**Why impressive:** The single most-requested feature in Shopify theme development. Every agency project needs one.

**Technical challenges:**
- Section Rendering API for cart drawer updates (`?section_id=cart-drawer`)
- Product Recommendations API (`/recommendations/products.json`)
- AJAX discount code application
- Cart line properties for gift wrapping
- Focus trap and scroll lock for slide-out panel
- Free shipping threshold calculation

**Complexity:** Medium (2-3 days)
**Where:** Add to Zenith theme as a section group in `layout/theme.liquid`

---

### Project 2: Predictive Search + Collection Filtering (Priority 2)

**What:** Full predictive search using `/search/suggest.json` and faceted collection filtering using the Storefront Filtering API.

**Why impressive:** Second most requested feature. Demonstrates JavaScript + Liquid integration and Shopify's data layer.

**Technical challenges:**
- Fetch-based predictive search with debounced input
- Grouped results (products, collections, pages, articles)
- `collection.filters` Liquid object for faceted filtering
- URL parameter management for filter state
- Section Rendering API for AJAX filter updates
- Price range slider
- Active filter tags with removal
- WAI-ARIA combobox pattern for search

**Complexity:** Medium-high (3-4 days)
**Where:** Add to any existing theme

---

### Project 3: Metaobject-Powered Content System (Priority 3)

**What:** "Store Locator" and "Team Members" content system built entirely on Shopify Metaobjects.

**Why impressive:** Shows data architecture skill. Avoids third-party CMSes. Most developers don't know Metaobjects well.

**Technical challenges:**
- Metaobject definition design (fields, validations, capabilities)
- `metaobjects` Liquid access with pagination
- Dynamic sources in section schema (`"type": "metaobject_reference"`)
- Google Maps embed from location fields
- Filtering/sorting metaobjects in Liquid

**Complexity:** Medium (2-3 days)
**Where:** Add to Zenith theme ("our gyms" + "our trainers")

**Implementation:**
- `store_location` metaobject: name, address, coordinates, phone, hours, image
- `team_member` metaobject: name, role, bio, photo, social_links
- `sections/store-locator.liquid` renders locations with map
- `sections/team-grid.liquid` renders team cards
- `templates/page.locations.json` and `templates/page.team.json`

---

### Project 4: Checkout Extension + Shopify Function (Priority 4)

**What:** Checkout customization with loyalty points display, gift message field, and volume discount function.

**Why impressive:** Checkout Extensions are the #1 most-requested Shopify development skill since checkout.liquid deprecation. Very few portfolio developers have working examples.

**Technical challenges:**
- Checkout UI Extension with React (`Checkout::Dynamic::Render` targets)
- Shopify Function in Rust or JavaScript for discount calculation
- App scaffolding with `@shopify/app` CLI
- Extension point targeting (order-summary, shipping, payment)
- Reading cart attributes and metafields in checkout context

**Complexity:** High (3-5 days, requires app scaffolding)
**Where:** Install as custom app on dev store

---

### Project 5: Performance Optimization Case Study (Priority 5)

**What:** Documented performance optimization pass on Zenith theme with before/after Lighthouse scores.

**Why impressive:** Separates junior from senior developers. Agencies billing $150+/hour expect 90+ Lighthouse scores.

**Technical challenges:**
- Critical CSS inlining and non-critical CSS deferral
- Image optimization (responsive srcset, loading="lazy", fetchpriority="high")
- JavaScript deferral and code splitting
- Font loading strategy (font-display: swap, preloading WOFF2)
- Section-level lazy loading
- DOM size reduction
- Before/after Lighthouse reports as artifacts

**Complexity:** Medium (2-3 days)
**Where:** Apply to `themes/starter-fitness`, document in `PERFORMANCE/`

---

### Project 6: Theme App Extension (Priority 6)

**What:** "Back in stock" notification widget (app block) + site-wide announcement system (app embed).

**Why impressive:** Shows app-to-theme integration model. Critical for app developers and agencies.

**Technical challenges:**
- App embed blocks (injected globally via theme editor)
- App blocks (added to specific sections/templates)
- App proxy for backend communication
- Session token authentication
- JavaScript SDK (`@shopify/app-bridge`)

**Complexity:** High (4-5 days, requires app scaffolding)
**Where:** Install as custom app, blocks appear in theme editor for all 6 themes

---

### Project 7: Multi-Language / Multi-Currency with Shopify Markets (Priority 7)

**What:** Full multi-language support with French + Spanish translations, language/currency selector, Shopify Markets configuration.

**Why impressive:** International commerce is Shopify's fastest-growing segment. Most developers have never implemented it.

**Technical challenges:**
- Complete locale file translations (`locales/fr.json`, `locales/es.json`)
- Schema locale files (`locales/fr.schema.json`)
- `form.localization` for country/language switcher
- `hreflang` link tags
- Currency formatting with `money_with_currency` filter

**Complexity:** Medium (2-3 days)
**Where:** Apply to one existing theme

---

## Recommended Build Order

| Order | Project | Type | Setup Needed |
|---|---|---|---|
| 1 | Cart Drawer with Upsells | Pure Liquid | None — add to existing theme |
| 2 | Predictive Search + Filtering | Pure Liquid + JS | None — add to existing theme |
| 3 | Metaobject Content System | Liquid + Admin | Metaobject definitions only |
| 4 | Checkout Extension + Function | App + Extension | `shopify app init` required |
| 5 | Performance Optimization | Analysis + Refactor | Lighthouse tooling |
| 6 | Theme App Extension | App + Extension | `shopify app init` required |
| 7 | Multi-Language / Markets | Locale files | Markets config in admin |

## Decision

**Start with projects 1-3** (pure Liquid, no app scaffolding needed) to fill the most visible portfolio gaps immediately. Then tackle project 4 (Checkout Extension) for the highest-impact differentiator.
