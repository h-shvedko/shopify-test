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

---

## Job Description Gap Analysis

**Source:** "Full Shopify Store Setup, Custom Apps & eCommerce Optimization" (analyzed 2026-03-18)

### Scope of Work Requirements

| # | Requirement | Coverage Status | Evidence / Notes |
|---|---|---|---|
| 1 | Full Shopify / Shopify Plus store setup (branding, layout, structure) | **Covered** | 6 themes built with distinct branding, layout systems, and page structures |
| 2 | Custom theme design and Shopify 2.0 theme development | **Covered** | Core competency — all 6 themes are OS 2.0 with JSON templates, sections, blocks |
| 3 | Advanced Liquid coding: custom sections, blocks, templates, reusable components | **Covered** | 300+ files across themes; sections, standalone blocks, snippets with `{% doc %}` |
| 4 | Product setup: collections, variants, inventory | **Covered** | Product templates, collection templates, variant selectors in all themes |
| 5 | B2B pricing logic | **Not Covered** | No wholesale pricing, quantity breaks, tiered pricing, or customer-tag-gated pricing |
| 6 | Secure payment gateway integration and checkout customization | **Partially Covered** | Project 4 (Checkout Extension) is proposed but not built; no payment gateway work |
| 7 | Custom app development | **Not Covered** | No custom apps exist; Projects 4 and 6 are proposed but not built |
| 8 | Third-party integrations | **Not Covered** | No API integrations, webhooks, or external service connections demonstrated |
| 9 | Mobile-first, responsive, SEO-friendly UX | **Partially Covered** | Mobile-first CSS is demonstrated; SEO is not explicitly showcased (no structured data, no sitemap customization, no meta tag strategy) |
| 10 | Page speed optimization | **Partially Covered** | Project 5 is proposed but not built; some lazy loading exists but no documented audit |
| 11 | Analytics setup and conversion tracking | **Not Covered** | No GA4 integration, no conversion pixel setup, no event tracking |
| 12 | Dropshipping setup | **Not Covered** | No dropshipping workflow, supplier integration, or fulfillment automation |
| 13 | Subscriptions | **Not Covered** | No subscription product setup, no recurring billing integration |
| 14 | Bundles | **Not Covered** | No product bundles, no bundle pricing logic |
| 15 | Membership setups | **Not Covered** | No gated content, no member-only pricing, no customer tag-based access |
| 16 | Ongoing maintenance and bug fixes | **Covered** | Demonstrated via iterative bug fix commits across all themes |
| 17 | Performance optimization | **Partially Covered** | Same as #10 |

### Deliverables

| # | Deliverable | Coverage Status | Notes |
|---|---|---|---|
| 1 | Fully designed and developed Shopify store ready to launch | **Covered** | 6 launch-ready themes on dev store |
| 2 | Custom apps or workflow automation | **Not Covered** | No apps, no Shopify Flow workflows |
| 3 | Documentation and training for store management | **Not Covered** | No merchant-facing documentation, no training materials, no handoff guides |
| 4 | Recommendations for growth, CRO, and performance | **Not Covered** | No CRO audit, no A/B test setup, no performance benchmark reports |

### Skills Required

| # | Skill | Coverage Status | Notes |
|---|---|---|---|
| 1 | Shopify and Shopify Plus development | **Partially Covered** | Shopify standard is covered; no Plus-specific features (Scripts, B2B, checkout.liquid migration) |
| 2 | Liquid | **Covered** | Strong demonstration across all themes |
| 3 | PHP | **Not Applicable** | PHP is not used in Shopify theme development; relevant only for app backends |
| 4 | JavaScript and API integrations | **Partially Covered** | Vanilla JS is strong; no external API integrations (REST or GraphQL to third parties) |
| 5 | eCommerce UX/UI design | **Covered** | 6 distinct design systems with varied aesthetics |
| 6 | SEO optimization for Shopify stores | **Not Covered** | No structured data (JSON-LD), no meta tag management, no SEO audit |
| 7 | Shopify apps, customizations, workflows | **Not Covered** | No custom apps, no Shopify Flow workflows |

### Optional Extras

| # | Extra | Coverage Status | Notes |
|---|---|---|---|
| 1 | Shopify Plus checkout customization | **Not Covered** | Project 4 proposed but not built |
| 2 | Multi-language / multi-currency setup | **Not Covered** | Project 7 proposed but not built |
| 3 | Advanced automation using Shopify Flow or private apps | **Not Covered** | No Flow workflows, no private apps |

### Summary

- **Covered:** 7 of 24 requirements (29%)
- **Partially Covered:** 5 of 24 requirements (21%)
- **Not Covered:** 11 of 24 requirements (46%)
- **Not Applicable:** 1 of 24 requirements (4%)

The existing portfolio strongly demonstrates theme development and Liquid expertise. The major gaps fall into four categories: **(A)** app development and integrations, **(B)** commerce logic (B2B, subscriptions, bundles), **(C)** operational deliverables (SEO, analytics, documentation), and **(D)** Plus-specific features.

---

## Additional Showcase Projects (Job-Driven)

These projects specifically target the gaps identified in the job description analysis above. They are numbered starting at 8 to continue from the existing project list.

### Project 8: B2B Wholesale Pricing System (Addresses Gap #5, #15)

**What:** Customer-tag-based wholesale pricing with tiered quantity breaks, minimum order quantities, and a gated wholesale registration form.

**Why impressive:** B2B is Shopify Plus's fastest-growing use case. Most freelancers cannot demonstrate wholesale logic. This directly addresses the "B2B pricing logic" and "membership setups" requirements.

**Implementation approach:**
- Customer tags (`wholesale`, `vip`, `distributor`) assigned via registration form or admin
- Liquid conditional rendering: `{% if customer.tags contains 'wholesale' %}` to show wholesale prices
- Metafield-stored tier pricing on products (`custom.wholesale_price`, `custom.tier_2_price`)
- Quantity break table on product page showing volume discounts
- Minimum order quantity enforcement in cart with JavaScript validation
- Wholesale-only collection page gated behind customer login
- Registration form that submits to a custom page template with approval workflow notes

**Complexity:** Medium (2-3 days)
**Where:** Add to BrewMaster theme (coffee wholesale is a natural fit)

---

### Project 9: Custom Shopify App — Order Dashboard (Addresses Gaps #7, #8)

**What:** A custom Shopify app with an embedded admin panel showing order analytics, a storefront widget via app embed, and webhook-driven order status notifications.

**Why impressive:** Directly proves "custom app development" and "third-party integrations." This is the single biggest gap in the portfolio and the #1 differentiator for senior Shopify developers.

**Implementation approach:**
- Scaffold with `shopify app init` (Node.js + React for admin, Remix template)
- Admin panel: embedded app showing order volume chart, top products, revenue by day
- Shopify Admin API (GraphQL): fetch orders, products, customers
- Webhooks: `orders/create`, `orders/paid` — log to app database
- App proxy: `/apps/dashboard` route serving storefront-accessible data
- Theme App Extension: app embed block showing "order status lookup" widget
- Session token authentication via `@shopify/shopify-app-remix`

**Complexity:** High (5-7 days)
**Where:** Install on dev store, app embed available in all themes

---

### Project 10: SEO Optimization Showcase (Addresses Gaps #9, #17)

**What:** Comprehensive SEO implementation across one theme with JSON-LD structured data, automated meta tags, breadcrumbs, canonical URLs, sitemap considerations, and a before/after audit report.

**Why impressive:** SEO is listed as a required skill in the job description. Most theme developers ignore structured data entirely. A documented audit with measurable results is strong proof.

**Implementation approach:**
- `snippets/structured-data.liquid`: JSON-LD for Product, Organization, BreadcrumbList, CollectionPage, Article, FAQPage schemas
- `snippets/seo-meta.liquid`: automated meta title/description with fallback hierarchy (SEO fields > product description > store name)
- Breadcrumb navigation component using `collection.url` hierarchy
- Canonical URL handling for paginated collections and filtered views
- Open Graph and Twitter Card meta tags for social sharing
- `robots.txt` customization notes
- Image `alt` tag audit and enforcement in schema settings (make image alt required)
- Before/after SEO audit document using Lighthouse SEO score, Google Rich Results Test, and Schema.org validator

**Complexity:** Medium (2-3 days)
**Where:** Apply to Zenith theme, document results in `SEO/` directory

---

### Project 11: Analytics and Conversion Tracking Setup (Addresses Gap #11)

**What:** Full analytics implementation with Shopify Customer Events API, GA4 integration, Meta Pixel, and custom conversion events — with a setup guide as a deliverable.

**Why impressive:** "Analytics setup and conversion tracking" is explicitly listed in the job scope. Most developers install apps for this; doing it natively shows deep platform knowledge.

**Implementation approach:**
- Shopify Customer Events (web pixels): custom pixel script for GA4 `purchase`, `add_to_cart`, `view_item`, `begin_checkout` events
- `snippets/analytics-events.liquid`: data layer population from Liquid objects
- GA4 Measurement Protocol integration for server-side events (via app proxy or webhook)
- Meta Pixel: standard events mapped to Shopify customer events
- Conversion tracking: thank-you page event firing with order value, items, transaction ID
- UTM parameter persistence through checkout
- Setup guide document: step-by-step for merchants to connect their GA4 property and verify events in DebugView

**Complexity:** Medium (2-3 days)
**Where:** Add to Zenith theme as reusable snippets, document in `ANALYTICS/`

---

### Project 12: Subscription and Bundle Product Setup (Addresses Gaps #13, #14)

**What:** Subscription selling plan integration for consumable products (e.g., protein powder, coffee) and a product bundle builder with combined pricing.

**Why impressive:** Subscriptions and bundles are explicitly required. These are revenue-driving features that clients ask for by name. Demonstrating native Shopify selling plans avoids third-party app dependency.

**Implementation approach:**
- **Subscriptions:** Selling plan group integration using the `selling_plan_group` Liquid object
  - Product page UI showing one-time vs. subscribe-and-save pricing
  - Selling plan selector (every 2 weeks / monthly / every 2 months)
  - Discount display (e.g., "Save 15% with subscription")
  - Cart line item showing subscription details
  - Requires selling plan creation via Admin API or app
- **Bundles:** Shopify Bundles app integration or custom bundle logic
  - "Build your box" section: select 3-5 products for a bundled price
  - Cart adjustment: bundle products submitted as a single cart item with line properties
  - Bundle discount calculation in cart
  - Bundle product card snippet showing component products

**Complexity:** Medium-high (3-4 days)
**Where:** Add subscriptions to BrewMaster (coffee subscriptions), bundles to PawHaven (pet care box)

---

### Project 13: Shopify Flow Automation Workflows (Addresses Gap #3 in Optional Extras)

**What:** A set of documented Shopify Flow workflows demonstrating automated operations: customer tagging, inventory alerts, order routing, and loyalty program triggers.

**Why impressive:** Shopify Flow is listed as an optional extra but having working examples immediately signals operational Shopify expertise, not just front-end skills.

**Implementation approach:**
- **Workflow 1 — Auto-tag VIP customers:** When order total exceeds threshold, tag customer as `vip`, send internal notification
- **Workflow 2 — Low inventory alert:** When variant inventory drops below 5, send Slack notification and add product tag `low-stock`
- **Workflow 3 — Order routing:** Tag orders by shipping country for fulfillment routing (domestic vs. international)
- **Workflow 4 — Win-back campaign trigger:** When customer has not ordered in 90 days, add tag `win-back` for email marketing segmentation
- **Workflow 5 — Review request:** 14 days after order fulfillment, tag customer for review request email sequence
- Export workflows as JSON, document each with trigger/condition/action diagrams
- Screenshot each workflow in Shopify admin for portfolio presentation

**Complexity:** Low-medium (1-2 days)
**Where:** Configure on dev store, document in `FLOWS/` directory

---

### Project 14: Merchant Documentation and Training Package (Addresses Gap — Deliverable #3)

**What:** A complete merchant handoff package for one theme: theme editor guide, content management guide, order processing guide, and video walkthrough script.

**Why impressive:** "Documentation and training for store management" is an explicit deliverable. Most developers skip this entirely. Having a professional handoff package shows client management maturity.

**Implementation approach:**
- **Theme Editor Guide:** Annotated screenshots showing how to customize each section, add/remove blocks, change colors and fonts, manage navigation menus
- **Content Management Guide:** How to create/edit products, collections, pages, blog posts, metaobjects; image sizing recommendations per section
- **Order Processing Guide:** Order fulfillment workflow, refund process, customer communication templates
- **FAQ Document:** Common questions (how to change the logo, how to add a new collection, how to hide a section)
- **Store Health Checklist:** Pre-launch checklist covering payment setup, shipping zones, tax configuration, legal pages, test order verification
- Format: Markdown documents convertible to PDF, organized in a `DOCS/` directory

**Complexity:** Low-medium (1-2 days)
**Where:** Create for Zenith theme in `DOCS/starter-fitness/`

---

## Updated Build Order (Job-Aligned)

Incorporating the new job-driven projects into the existing build order, re-prioritized by impact on this specific job application:

| Priority | Project | Type | Effort | Job Requirement Addressed |
|---|---|---|---|---|
| 1 | Project 1: Cart Drawer with Upsells | Liquid + JS | 2-3 days | High-converting UX |
| 2 | Project 10: SEO Optimization Showcase | Liquid + Audit | 2-3 days | SEO optimization (required skill) |
| 3 | Project 9: Custom Shopify App | App + Admin API | 5-7 days | Custom app development (scope of work) |
| 4 | Project 8: B2B Wholesale Pricing | Liquid + Metafields | 2-3 days | B2B pricing logic (scope of work) |
| 5 | Project 12: Subscriptions and Bundles | Liquid + Selling Plans | 3-4 days | Subscriptions, bundles (scope of work) |
| 6 | Project 11: Analytics and Tracking | Pixels + GA4 | 2-3 days | Analytics, conversion tracking (scope of work) |
| 7 | Project 14: Merchant Documentation | Markdown/PDF | 1-2 days | Documentation and training (deliverable) |
| 8 | Project 13: Shopify Flow Automation | Flow + Docs | 1-2 days | Automation (optional extra) |
| 9 | Project 2: Predictive Search + Filtering | Liquid + JS | 3-4 days | High-converting UX |
| 10 | Project 4: Checkout Extension | App + Extension | 3-5 days | Checkout customization (optional extra) |
| 11 | Project 7: Multi-Language / Markets | Locales | 2-3 days | Multi-language/currency (optional extra) |
| 12 | Project 3: Metaobject Content System | Liquid + Admin | 2-3 days | Content modeling |
| 13 | Project 5: Performance Optimization | Analysis | 2-3 days | Page speed (scope of work) |
| 14 | Project 6: Theme App Extension | App + Extension | 4-5 days | App integrations |

**Estimated total effort for top 8 (job-critical) projects:** 19-27 days

**Recommendation:** For this specific job application, prioritize Projects 1, 10, 9, and 8 (approximately 12-16 days of work). These four projects would move portfolio coverage from 29% to approximately 63% of the job requirements, addressing the most visible gaps: custom app, SEO, B2B pricing, and polished UX.
