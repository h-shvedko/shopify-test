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

---

## Additional Showcase Projects (Remaining Gaps)

These projects address the last uncovered requirements from the gap analysis.

### Project 15: Dropshipping Fulfillment Integration (Addresses Gap #12)

**What:** A dropshipping-ready product setup with supplier integration via webhook/API, automated fulfillment routing, and inventory sync demonstration.

**Why impressive:** Dropshipping is explicitly listed in the scope of work. It's a high-volume business model on Shopify, and demonstrating the technical setup (not just installing an app) shows operational depth.

**Implementation approach:**
- **Product architecture:** Products with `custom.supplier` metafield and `custom.supplier_sku` for mapping to supplier catalogs
- **Fulfillment workflow:** Shopify Flow workflow that routes orders to different fulfillment services based on product vendor/tag
- **Inventory sync concept:** Webhook listener (`inventory_levels/update`) that demonstrates how supplier stock feeds into Shopify inventory
- **Supplier management page:** Admin metaobject `supplier` with fields: name, API endpoint, lead time, shipping origin country
- **Automated order forwarding:** Documented webhook flow — `orders/create` → parse line items → forward supplier-specific items to supplier API endpoint
- **Shipping estimate snippet:** `snippets/shipping-estimate.liquid` that calculates and displays estimated delivery based on supplier lead time metafield
- **Documentation:** Supplier onboarding guide, fulfillment SLA expectations, inventory sync frequency recommendations

**Complexity:** Medium (2-3 days)
**Where:** Add to PawHaven theme (pet supplies dropshipping is a common use case)

---

### Project 16: CRO Audit and Growth Recommendations (Addresses Deliverable #4)

**What:** A complete Conversion Rate Optimization audit of one theme with documented findings, A/B test hypotheses, and actionable growth recommendations delivered as a professional report.

**Why impressive:** "Recommendations for growth, CRO, and performance improvements" is an explicit deliverable. This proves consulting-level thinking beyond code execution — the ability to advise merchants on revenue growth.

**Implementation approach:**
- **Funnel analysis document:** Map the customer journey (homepage → collection → product → cart → checkout) with conversion drop-off points and optimization recommendations at each stage
- **Heatmap-informed UX audit:** Document above-the-fold content hierarchy, CTA placement, visual hierarchy issues, and scroll depth assumptions for each key page
- **A/B test hypotheses:** 10 documented test ideas with hypothesis format ("If we [change], then [metric] will [improve] because [reason]"):
  1. Hero CTA copy and color contrast
  2. Product page image gallery layout (carousel vs. grid)
  3. Add-to-cart button placement (sticky vs. static)
  4. Social proof positioning (reviews above vs. below fold)
  5. Free shipping threshold bar in header
  6. Collection page grid density (3-col vs. 4-col)
  7. Cart upsell recommendation algorithm (related vs. complementary)
  8. Mobile navigation pattern (hamburger vs. bottom tabs)
  9. Product description format (tabs vs. accordion vs. long-form)
  10. Urgency elements (stock countdown, limited-time badge)
- **Quick wins implementation:** Implement 3-5 quick CRO improvements directly in theme code:
  - Trust badges below add-to-cart button (`snippets/trust-badges.liquid`)
  - Sticky add-to-cart bar on mobile for product pages
  - "Customers also bought" section on cart page using Product Recommendations API
  - Exit-intent newsletter popup (vanilla JS, `mouseleave` event on desktop)
- **Growth strategy document:** Recommendations for email capture, retention marketing, AOV improvement, and repeat purchase strategies
- **Deliverable format:** Professional PDF-ready Markdown report in `CRO/` directory with screenshots, annotated mockups, and prioritized action items

**Complexity:** Medium (2-3 days)
**Where:** Audit Zenith theme, implement quick wins, document in `CRO/`

---

### Project 17: Merchant Training Video Scripts and Knowledge Base (Extends Project 14)

**What:** Video walkthrough scripts and a Shopify-native knowledge base page template that merchants can use for self-service store management.

**Why impressive:** Goes beyond static documentation to demonstrate client enablement skills. Agencies charge $2,000-5,000 for training packages. Having scripts ready shows productized service capability.

**Implementation approach:**
- **Video script series** (5 scripts, each 3-5 minutes):
  1. "Customizing your homepage" — theme editor walkthrough for sections and blocks
  2. "Managing products and collections" — bulk editing, variants, images, SEO fields
  3. "Processing orders" — fulfillment, refunds, draft orders, notes
  4. "Updating your navigation and pages" — menus, link lists, page templates
  5. "Checking store health" — Shopify analytics overview, speed report, common issues
- **Knowledge base page template:** `templates/page.help-center.json` with an FAQ section, search functionality (reuse FAQ search from Zenith), and categorized help articles using metaobjects
- **Help center metaobject:** `help_article` with fields: title, category, body (rich text), related_articles, difficulty_level
- **Quick-start checklist snippet:** `snippets/merchant-checklist.liquid` for embedding an interactive launch checklist on a staff-only page

**Complexity:** Low-medium (1-2 days)
**Where:** Scripts in `DOCS/training-scripts/`, knowledge base template in Zenith theme

---

## Final Updated Build Order (Complete Job Coverage)

Full prioritized list including all 17 projects, optimized for maximum job requirement coverage:

| Priority | Project | Type | Effort | Job Requirement | Coverage Impact |
|---|---|---|---|---|---|
| 1 | P1: Cart Drawer with Upsells | Liquid + JS | 2-3d | High-converting UX | Fills most-requested feature gap |
| 2 | P10: SEO Optimization | Liquid + Audit | 2-3d | SEO (required skill) | Fills explicit skill requirement |
| 3 | P9: Custom Shopify App | App + Admin API | 5-7d | Custom apps (scope) | Fills biggest portfolio gap |
| 4 | P8: B2B Wholesale Pricing | Liquid + Metafields | 2-3d | B2B pricing (scope) | Fills Shopify Plus signal |
| 5 | P12: Subscriptions + Bundles | Liquid + Selling Plans | 3-4d | Subscriptions, bundles (scope) | Fills 2 scope items at once |
| 6 | P11: Analytics + Tracking | Pixels + GA4 | 2-3d | Analytics (scope) | Fills explicit scope item |
| 7 | P16: CRO Audit + Recommendations | Audit + Quick Wins | 2-3d | Growth/CRO (deliverable) | Fills explicit deliverable |
| 8 | P14: Merchant Documentation | Markdown/PDF | 1-2d | Training (deliverable) | Fills explicit deliverable |
| 9 | P13: Shopify Flow Automation | Flow + Docs | 1-2d | Automation (optional) | Fills optional extra |
| 10 | P15: Dropshipping Integration | Metafields + Flow | 2-3d | Dropshipping (scope) | Fills explicit scope item |
| 11 | P2: Predictive Search + Filtering | Liquid + JS | 3-4d | High-converting UX | Enhances theme sophistication |
| 12 | P4: Checkout Extension | App + Extension | 3-5d | Checkout (optional) | Fills Plus-specific gap |
| 13 | P7: Multi-Language / Markets | Locales | 2-3d | Multi-lang (optional) | Fills optional extra |
| 14 | P17: Training Videos + KB | Scripts + Template | 1-2d | Training (deliverable) | Enhances deliverable #3 |
| 15 | P3: Metaobject Content System | Liquid + Admin | 2-3d | Content modeling | Enhances data architecture |
| 16 | P5: Performance Optimization | Analysis | 2-3d | Page speed (scope) | Fills performance gap |
| 17 | P6: Theme App Extension | App + Extension | 4-5d | App integrations | Advanced app development |

### Coverage Projection

| Milestone | Projects Completed | Estimated Coverage | Effort |
|---|---|---|---|
| Current state | None (6 themes only) | 29% covered, 21% partial | — |
| After top 4 | P1, P10, P9, P8 | ~58% covered | 12-16 days |
| After top 8 | + P12, P11, P16, P14 | ~79% covered | 22-31 days |
| After top 12 | + P13, P15, P2, P4 | ~92% covered | 32-44 days |
| All 17 | Complete | ~96% covered (PHP remains N/A) | 42-58 days |

### Job Application Strategy

**Immediate actions (before applying):**
1. Complete Project 1 (Cart Drawer) — fastest way to show polished UX skill
2. Complete Project 10 (SEO) — directly addresses a listed required skill
3. Prepare a 1-page portfolio summary highlighting the 6 themes with live preview links

**In the proposal, emphasize:**
- 6 production-ready themes covering diverse verticals (boutique → fitness → luxury → pets)
- OS 2.0 architecture mastery with 300+ Liquid files
- Accessibility compliance (WCAG 2.1) across all themes
- Zero framework dependency — vanilla JS, no jQuery
- Advanced features already built: quick-view, product comparison, multi-step cart, mega menu, dark mode toggle

**Address gaps honestly:**
- "Custom app development is my next focus area — I have the architecture planned and am building a dashboard app with Admin API integration"
- "I've implemented mobile-first responsive design across 6 themes and am adding structured data/SEO optimization to demonstrate the full stack"

**Differentiator:** Most applicants will have 1-2 themes. Having 6 distinct themes with varied design systems (minimal, editorial, terminal, luxury, natural, cinematic) demonstrates range and speed that is hard to match.

---

## Job Description Gap Analysis #2: Senior Shopify App Developer — Compliance SaaS Integration

**Source:** "Senior Shopify App Developer — Compliance SaaS Integration (GraphQL, React, Node.js)" (analyzed 2026-03-19)

**Role summary:** Build a public Shopify app for a B2B compliance SaaS company (sanctions screening, PEP data, adverse media monitoring). The app screens merchants' orders and customers against global sanctions lists (OFAC, EU, UN, HM Treasury) directly from Shopify admin. Long-term engagement with 10-20 hours/month maintenance after launch.

### Requirements Coverage

| # | Requirement | Coverage Status | Notes |
|---|---|---|---|
| 1 | Public Shopify App (App Store distribution) | **Not Covered** | No published app. Project 9 is a private/custom app concept. |
| 2 | Embedded app (Shopify App Bridge + Polaris) | **Not Covered** | No Polaris usage anywhere. App Bridge mentioned in P6 but not built. |
| 3 | GraphQL Admin API (exclusively) | **Not Covered** | Mentioned in P9 but not implemented. Job explicitly rejects REST-based proposals. |
| 4 | OAuth install flow + session token auth | **Not Covered** | P9 mentions session tokens but no OAuth handshake, consent screen, or token storage. |
| 5 | Shopify Billing API (subscription + free trial) | **Not Covered** | Not mentioned in any project. |
| 6 | GDPR webhooks (mandatory for App Store) | **Not Covered** | `customers/data_request`, `customers/redact`, `shop/redact` — not mentioned. |
| 7 | Automated account provisioning on install | **Not Covered** | No external SaaS account creation flow. |
| 8 | Order screening via webhooks | **Partially Covered** | P9 mentions `orders/create` webhook but no screening/processing logic. |
| 9 | Customer screening (on-demand + automatic) | **Not Covered** | No screening concept in any project. |
| 10 | Manual screening UI panel in admin | **Not Covered** | No interactive data table or search UI in embedded app context. |
| 11 | Match review UI (risk levels, false positive marking) | **Not Covered** | No review workflow or risk assessment UI. |
| 12 | Settings panel (sensitivity, thresholds, list selection) | **Not Covered** | No configurable app settings UI. |
| 13 | Webhook reliability (idempotency, retries, queues) | **Not Covered** | Listed as nice-to-have but strongly preferred. No reliability patterns anywhere. |
| 14 | External SaaS API integration | **Not Covered** | No API integration with third-party services. |
| 15 | App Store submission + review process | **Not Covered** | No App Store experience. |
| 16 | Built for Shopify certification | **Not Covered** | No awareness of BFS requirements documented. |
| 17 | React + TypeScript frontend | **Not Covered** | ADR states "vanilla JS only, no frameworks." |
| 18 | Node.js backend/middleware | **Not Covered** | No backend project exists. |
| 19 | Shopify Flow integration (Phase 2+) | **Partially Covered** | P13 proposes Flow workflows but not custom triggers/actions from an app. |
| 20 | Ongoing maintenance + API version upgrades | **Covered** | Demonstrated via iterative commits across themes. |

### Summary

- **Covered:** 1 of 20 requirements (5%)
- **Partially Covered:** 2 of 20 requirements (10%)
- **Not Covered:** 17 of 20 requirements (85%)

**Core mismatch:** ADR-0002 has 15 of 17 projects focused on theme development. This job is 100% app development. The tech stack gap is fundamental — the job requires React, TypeScript, Node.js, Polaris, GraphQL, and OAuth, none of which are demonstrated in the current portfolio.

---

## Additional Showcase Projects (App Developer Track)

These projects specifically target the Shopify App Developer gap. They are numbered starting at 18 to continue from the existing project list.

### Project 18: Public Shopify App — Compliance Screening Tool (HIGHEST PRIORITY)

**What:** A full public Shopify app that screens orders and customers against an external sanctions/compliance API, with an embedded Polaris admin panel, OAuth install flow, Shopify Billing API integration, GDPR webhook compliance, and webhook reliability patterns. Target: App Store submission.

**Why impressive:** This single project addresses 17 of the 20 identified gaps simultaneously. It is the job description implemented as a portfolio piece. Building this before applying transforms a theme-only portfolio into a credible full-stack Shopify developer portfolio.

**Implementation approach:**

- **App scaffold:** `shopify app init` with the Remix template (Node.js + React + TypeScript)
- **OAuth install flow:** Full merchant authorization with required scopes (`read_orders`, `read_customers`), offline access tokens stored encrypted at rest in database
- **Shopify Billing API:** `appSubscriptionCreate` GraphQL mutation, monthly recurring charge with 14-day free trial, usage-based billing tier for screening volume, grace period handling for failed charges
- **Embedded admin UI (Polaris + App Bridge):**
  - **Dashboard page:** screening stats, recent matches, risk level breakdown (high/medium/low/clear), activity feed
  - **Order screening page:** data table of screened orders with risk badges, detail view with match reasons, false positive marking, bulk actions
  - **Customer screening page:** on-demand screening trigger, screening history per customer, risk score timeline
  - **Manual screening page:** ad-hoc name/entity search form, results with match confidence scores
  - **Match review page:** detailed match display with entity comparison (name similarity, country, date of birth), approve/reject workflow, audit log with timestamps
  - **Settings page:** screening sensitivity slider, notification email configuration, auto-hold threshold, API key display, list selection (OFAC, EU, UN, HM Treasury checkboxes)
- **Webhook handling:**
  - `orders/create` and `orders/updated` → trigger automatic screening
  - `customers/create` → trigger customer screening
  - `app/uninstalled` → cleanup routine (suspend backend account, revoke API key)
  - GDPR mandatory: `customers/data_request`, `customers/redact`, `shop/redact`
  - **Idempotency:** deduplicate by `X-Shopify-Webhook-Id` header + event timestamp
  - **HMAC verification:** validate `X-Shopify-Hmac-Sha256` for all incoming webhooks
  - **Queue-based processing:** webhook receipt returns 200 immediately; processing via BullMQ job queue with Redis
  - **Retry handling:** exponential backoff with configurable max retries
  - **Dead letter queue:** capture permanently failed webhook processing for manual review
- **External SaaS integration:** Mock compliance API (or OpenSanctions open data) demonstrating: API key management, request/response mapping, rate limiting, error handling with circuit breaker pattern
- **Automated account provisioning:** On app install → create backend account for merchant → generate API credentials → store securely → provision default screening config → surface API key in settings UI
- **Session token auth:** App Bridge session token validation middleware, no cookie-based sessions
- **Database:** Prisma ORM with SQLite (dev) / PostgreSQL (production) — tables: merchants, api_keys, screening_results, audit_log, webhook_events
- **App Store preparation:** listing description, feature screenshots, privacy policy, data handling disclosure

**Tech stack:** Node.js, React (TypeScript), Remix, Prisma, BullMQ + Redis, Polaris, App Bridge, GraphQL Admin API

**Complexity:** High (10-14 days)
**Where:** Standalone app repository, installed on dev store

---

### Project 19: Webhook Reliability Patterns Library (PRIORITY 2)

**What:** A documented, reusable Node.js module demonstrating production-grade webhook handling patterns, packaged as a reference implementation within the app project.

**Why impressive:** Webhook reliability (idempotency, retries, queue-based processing) is what separates a junior app developer from a senior one. The job lists it as nice-to-have but in practice it's a strong signal of production experience.

**Implementation approach:**
- **Idempotency middleware:** Deduplicate webhooks by `X-Shopify-Webhook-Id` header, backed by a `webhook_events` table with unique constraint
- **HMAC verification middleware:** Validate `X-Shopify-Hmac-Sha256` using app secret, reject invalid signatures
- **Queue-based processing:** BullMQ job queue separating webhook receipt (fast 200 response) from processing (async worker). Configurable concurrency and rate limiting.
- **Retry with exponential backoff:** Configurable strategy (initial delay, max retries, backoff multiplier) for failed processing
- **Dead letter queue:** Capture permanently failed jobs for manual review with full payload and error context
- **Circuit breaker:** For outbound API calls to external SaaS — prevent cascade failures when the compliance API is down
- **Structured logging:** JSON logs with correlation IDs, processing duration, webhook type, and outcome
- **Test suite:** Unit tests for each pattern with mock Shopify webhook payloads

**Complexity:** Medium (3-4 days, can be built as part of Project 18)
**Where:** Module within Project 18's app codebase

---

### Project 20: Shopify Flow Custom Trigger and Action (PRIORITY 3)

**What:** Custom Flow trigger ("Screening match found") and custom Flow action ("Screen customer on demand") exposed by the compliance app.

**Why impressive:** The job lists Shopify Flow as a Phase 2+ integration. Custom triggers/actions (not just using built-in Flow) show deep platform integration capability beyond theme-level Flow workflows in P13.

**Implementation approach:**
- **Custom Flow trigger — "Screening match found":** Fires when a customer or order matches a sanctions list entry. Passes: risk level, match confidence score, matched list name, entity details. Merchants can build flows like: "When high-risk match found → tag order as held → send Slack notification."
- **Custom Flow action — "Screen customer on demand":** Merchants can build flows that trigger screening. Example: "When customer tag added 'needs-review' → screen customer against all lists."
- Extension configuration in `shopify.app.toml` with `flow` extension targets
- Documentation of 3-5 example workflows merchants can compose

**Complexity:** Medium (2-3 days, requires Project 18 as foundation)
**Where:** Extension within Project 18's app

---

### Project 21: Built for Shopify Certification Checklist

**What:** A documented checklist covering all Built for Shopify (BFS) certification requirements, applied to Project 18 as a compliance target.

**Why impressive:** The job explicitly requires Built for Shopify certification. Having a documented checklist shows awareness of the program even before completing certification.

**Checklist content:**
- App performance: API rate limit compliance, webhook response time under 5 seconds
- Required GDPR webhooks implemented and tested
- App listing quality: description, screenshots, support URL, privacy policy, data handling disclosure
- Session token auth (no cookie-based auth, no redirect flows)
- Billing API usage (no external payment collection for app charges)
- Embedded app requirement (no redirect-based admin flows)
- Polaris usage for all admin UI components
- Accessibility compliance in admin UI (Polaris handles most, but custom components need audit)
- App health metrics: install rate, crash rate, uninstall rate targets
- Shopify API version currency (not more than 2 versions behind)

**Complexity:** Low (0.5-1 day to document, ongoing to maintain)
**Where:** Document in `APP/built-for-shopify-checklist.md`

---

## Revised Build Order (App Developer Track)

For the Compliance SaaS App Developer role, the build order must pivot from theme-centric to app-centric projects:

| Priority | Project | Type | Effort | Coverage Impact |
|---|---|---|---|---|
| **1** | **P18: Compliance Screening App** | App (React/Node/GraphQL) | 10-14d | Closes 80% of gaps alone |
| **2** | **P19: Webhook Reliability Patterns** | Node.js module | 3-4d | Demonstrates production maturity |
| **3** | **P20: Shopify Flow Custom Trigger/Action** | App extension | 2-3d | Demonstrates platform depth |
| **4** | **P21: Built for Shopify Checklist** | Documentation | 0.5-1d | Shows certification awareness |
| 5 | P4: Checkout Extension + Function | App + Extension | 3-5d | Shows extension ecosystem breadth |
| 6 | P6: Theme App Extension | App + Extension | 4-5d | Shows app-to-theme integration |
| 7 | P13: Shopify Flow Automation | Flow + Docs | 1-2d | Supports P20 with merchant workflows |

### Coverage Projection (Compliance SaaS Job)

| Milestone | Projects Completed | Coverage | Effort |
|---|---|---|---|
| Current state | None (6 themes only) | ~5% covered | — |
| After P18 | Compliance Screening App | ~80% covered | 10-14 days |
| After P18 + P19 | + Webhook Reliability | ~88% covered | 13-18 days |
| After P18-P20 | + Flow Integration | ~93% covered | 15-21 days |
| After P18-P21 | + BFS Checklist | ~95% covered | 16-22 days |
| All app-track projects | + P4, P6, P13 | ~97% covered | 24-34 days |

### What to Deprioritize for This Role

These existing projects have **low relevance** for an app developer role and should be deferred:

| Project | Reason |
|---|---|
| P1: Cart Drawer | Theme-only; this job has no theme development |
| P2: Predictive Search | Theme-only |
| P3: Metaobjects | Theme-only |
| P5: Performance Optimization | Theme-only (app performance is different) |
| P7: Multi-Language / Markets | Theme-only |
| P8: B2B Wholesale Pricing | Theme-only |
| P10: SEO Optimization | Theme-only |
| P14: Merchant Documentation | Theme-only |
| P15: Dropshipping Integration | Theme-only |
| P16: CRO Audit | Theme-only |
| P17: Training Videos | Theme-only |

These existing projects **remain relevant:**

| Project | Relevance |
|---|---|
| P4: Checkout Extension | Demonstrates React in Shopify extension context |
| P6: Theme App Extension | Demonstrates app blocks, app embeds, App Bridge |
| P9: Custom Shopify App | Should be merged into P18 (P18 is a superset) |
| P13: Shopify Flow | Foundation for P20's custom triggers/actions |

---

## Job Application Strategy (Compliance SaaS Role)

### Before Applying

1. **Build Project 18** to at minimum a working demo: OAuth install, Polaris admin UI with screening dashboard, webhook processing, Billing API subscription creation. Even without App Store publication, a working demo with a screen recording is compelling.
2. **Extract webhook patterns** (Project 19) as a documented module — shows engineering rigor.
3. **Prepare a technical architecture diagram:** Shopify webhook → BullMQ queue → compliance API → risk assessment → Polaris UI → merchant action.

### In the Proposal

**Automated account provisioning approach** (the job asks for a paragraph on this):
> "On app install, the OAuth callback handler triggers an async provisioning job: (1) create a sanctions.io account via the REST API using the merchant's shop domain and email, (2) receive and encrypt the API key, storing it in the app database scoped to the merchant's `shop_id`, (3) set default screening configuration (all lists enabled, medium sensitivity), (4) surface the API key and account status in the Polaris settings page. Edge cases: if the merchant's email already has an account, link to existing; if provisioning fails, show a retry button with error context; on re-install, check for existing suspended account and reactivate."

**Recommended hosting approach:**
> "Fly.io for the Node.js middleware (auto-scaling, global edge deployment, built-in Redis for BullMQ), with PostgreSQL on Fly Postgres or Neon for the lightweight database. Alternative: Railway for simpler deployment with similar capabilities. The app is stateless except for the job queue, so horizontal scaling is straightforward."

### Frame the Theme Portfolio as an Advantage

> "My 6 Shopify themes demonstrate deep platform knowledge — I understand the merchant experience, the theme editor, the Liquid rendering pipeline, and Shopify's data model at a level most app developers do not. This translates directly to building better embedded app experiences because I know what merchants expect from their admin, how orders flow, and what the checkout looks like. Most app developers learn Shopify from the API docs. I learned it from building the storefront up."

### Key Differentiator

Most Shopify app developers come from a React/Node.js background and learn Shopify's API layer. A developer coming from deep Shopify platform knowledge (6 themes, OS 2.0 mastery, AJAX Cart API, section rendering) who adds app development skills has a fundamentally better understanding of the merchant context. Frame this as: "I build apps for merchants I already understand."
