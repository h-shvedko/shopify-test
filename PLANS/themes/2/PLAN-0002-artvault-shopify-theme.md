# PLAN-0002: ArtVault Starter Bold Shopify Theme Implementation

> Status: APPROVED
> Created: 2026-03-16
> Source: Figma file `g2fn6NTKAz2GtmEyNYxMUE`, design HTML `PLANS/themes/2/design-bold-theme.html`

## Context

The Figma design and HTML mockup describe **ArtVault** -- an art/creative studio e-commerce theme targeting galleries, artist marketplaces, and creative studios. This is the second theme in the project (`starter-bold`), following the `starter-minimal` BrewMaster theme.

**Key challenge:** The design is dense and immersive with editorial layouts, full-bleed grids, glassmorphism header, marquee animations, Quick View modals, and social feed sections. These go significantly beyond starter-minimal's straightforward grid layouts.

**Lessons learned from starter-minimal (BrewMaster):**

1. Use `<script>` tags instead of `{% javascript %}` (the Liquid tag does not render)
2. Use `{% content_for 'blocks' %}` for standalone blocks from the `blocks/` directory
3. Section schema must use `"type": "@theme"` for standalone block references
4. Move shared CSS (like `.btn`) to `critical.css` to avoid injection issues
5. Pre-assign translated strings to variables before passing to `{% render %}` tags
6. Snippets use raw `<style>` tags (not `{% stylesheet %}`)

---

## Design Summary

- **Brand:** ArtVault -- art and creative studio marketplace
- **Color scheme:** Forest green (#2D5A3D) primary, warm wood/copper (#C8956C) accent, off-white (#FAFAF7) background, near-black (#1A1A18) dark surfaces
- **Typography:** Playfair Display (serif) for headings with italic accent words, Inter (sans-serif) for body
- **Vibe:** Fun and energetic, dense and immersive, elegant serif typography
- **Pages:** Home, Product Detail, Cart (plus standard: collection, search, 404, page, customers)
- **Unique design elements:** Glassmorphism sticky header, marquee text strip, editorial 3-column collection grid (first item spans 2 rows), Quick View hover overlay on products, social/Instagram feed grid, testimonial cards, newsletter with image overlay

---

## Design Tokens

| Token | Value | Usage |
|---|---|---|
| **Colors** | | |
| Primary | `#2D5A3D` | Buttons, accents, active states |
| Primary Light | `#3A7A52` | Hover states |
| Primary Dark | `#1E3D2A` | Editorial bg, overlay gradients |
| Accent | `#C8956C` | CTA buttons, accent badges, stars |
| Accent Light | `#E8C4A0` | Logo italic, hero italic text, stat numbers |
| Accent Dark | `#A07048` | Category labels |
| Background | `#FAFAF7` | Page background |
| Background Alt | `#F0EDE8` | Testimonials bg, cart summary bg |
| Background Dark | `#1A1A18` | Header, footer, marquee |
| Background Warm | `#F5F0EB` | Products section bg |
| Text | `#1A1A18` | Body text |
| Text Secondary | `#6B6B63` | Descriptions |
| Text Muted | `#9B9B8F` | Meta text, placeholders |
| Text Light | `#F0EDE8` | Text on dark backgrounds |
| Border | `#E0DDD6` | Card borders, dividers |
| Error/Sale | `#C44536` | Sale badges, remove links |
| Wood | `#8B6F47` | Accent variant |
| **Typography** | | |
| Heading font | Playfair Display, Georgia, serif | All headings, logo |
| Body font | Inter, -apple-system, sans-serif | Body, labels, buttons |
| H1 size | 4-5rem (80px on hero) | Hero heading |
| H2 size | 2.75rem (44px) | Section headings |
| H3 size | 1.5rem (24px) | Card headings, sub-sections |
| Label style | 0.8rem, uppercase, letter-spacing 0.08em, weight 600 | Category labels, option labels |
| **Layout** | | |
| Container max-width | 1400px | Content wrapper |
| Border radius small | 4px | Cards, buttons |
| Border radius large | 8px | Larger cards, main image |
| Breakpoint tablet | 768px | `@media (min-width: 768px)` |
| Breakpoint desktop | 1024px | `@media (min-width: 1024px)` |
| Breakpoint wide | 1200px | `@media (min-width: 1200px)` |
| Collection grid gap | 8px | Dense editorial feel |
| Product grid gap | 24px | Breathing room for cards |
| Social grid gap | 4px | Edge-to-edge mosaic |
| Section vertical padding | 80px desktop / 48px mobile | All major sections |

---

## Implementation Approach

### Phase 0: Scaffold Directory Structure

Create `themes/starter-bold/` with all required subdirectories:
`assets/`, `blocks/`, `config/`, `layout/`, `locales/`, `sections/`, `snippets/`, `templates/`, `templates/customers/`

**Done when:** All directories exist and are empty.

---

### Phase 1: Foundation (config, layout, assets, locales)

**Files to create:**

#### 1. `config/settings_schema.json`

Global theme settings. Same structure as starter-minimal but with ArtVault's color palette, font choices, and wider container.

**Group: Theme Info**

| Field | Value |
|---|---|
| `theme_name` | `ArtVault Starter Bold` |
| `theme_version` | `1.0.0` |
| `theme_author` | `ArtVault` |

**Group: Colors**

| ID | Type | Default | Purpose |
|---|---|---|---|
| `color_primary` | `color` | `#2D5A3D` | Forest green primary |
| `color_primary_light` | `color` | `#3A7A52` | Hover/active states |
| `color_primary_dark` | `color` | `#1E3D2A` | Deep accents, editorial bg |
| `color_primary_contrast` | `color` | `#ffffff` | Text on primary bg |
| `color_accent` | `color` | `#C8956C` | Warm accent (wood/copper) |
| `color_accent_light` | `color` | `#E8C4A0` | Accent text on dark bg |
| `color_accent_dark` | `color` | `#A07048` | Category labels |
| `color_background` | `color` | `#FAFAF7` | Page background |
| `color_background_alt` | `color` | `#F0EDE8` | Alternate section bg |
| `color_background_dark` | `color` | `#1A1A18` | Header, footer, marquee |
| `color_background_warm` | `color` | `#F5F0EB` | Products section bg |
| `color_text` | `color` | `#1A1A18` | Body text |
| `color_text_secondary` | `color` | `#6B6B63` | Descriptions |
| `color_text_muted` | `color` | `#9B9B8F` | Meta text |
| `color_border` | `color` | `#E0DDD6` | Borders, dividers |
| `color_error` | `color` | `#C44536` | Error states, sale badges |
| `color_success` | `color` | `#2D5A3D` | Success states (same as primary) |

**Group: Typography**

| ID | Type | Default |
|---|---|---|
| `font_heading` | `font_picker` | `playfair_display_n7` |
| `font_body` | `font_picker` | `inter_n4` |
| `font_scale` | `range` | 100 (min: 80, max: 130, step: 5) |

**Group: Layout**

| ID | Type | Default |
|---|---|---|
| `page_width` | `range` | 1400 (min: 1200, max: 1600, step: 40) |
| `border_radius` | `range` | 4 (min: 0, max: 16, step: 2) |
| `border_radius_large` | `range` | 8 (min: 0, max: 24, step: 2) |
| `spacing_scale` | `range` | 100 (min: 80, max: 130, step: 10) |

**Group: Cart**

| ID | Type | Default |
|---|---|---|
| `free_shipping_threshold` | `number` | 99 |
| `show_free_shipping_bar` | `checkbox` | true |

**Group: Social**

| ID | Type | Default |
|---|---|---|
| `social_instagram` | `text` | `""` |
| `social_twitter` | `text` | `""` |
| `social_facebook` | `text` | `""` |
| `social_pinterest` | `text` | `""` |

#### 2. `config/settings_data.json`

Minimal default values pointing to schema defaults.

#### 3. `assets/critical.css`

CSS reset, `:root` variable fallbacks (all design token colors), `.container`, `.visually-hidden`, base typography (Playfair + Inter), button base classes (`.btn`, `.btn--primary`, `.btn--outline`, `.btn--dark`, `.btn--green`, `.btn--full`), label utility class. Keep under 8KB.

**Key difference from starter-minimal:** Button variants are richer (primary uses accent color, not green; outline is white border for dark backgrounds; green variant for add-to-cart and checkout). The `.btn` class must be in critical.css per lesson #4.

#### 4. `snippets/css-variables.liquid`

Renders a `<style>` tag with `:root` CSS custom properties mapped from `settings` object. Includes all color, typography, layout, and computed variables. Uses raw `<style>` element (lesson #6).

#### 5. `layout/theme.liquid`

Master layout: `<html>` with locale, `<head>` (meta, Google Fonts preconnect for Playfair Display + Inter, critical.css, css-variables snippet, `content_for_header`), `<body>` with skip-to-content link, `{% sections 'header-group' %}`, `<main>{{ content_for_layout }}</main>`, `{% sections 'footer-group' %}`

**Note:** Google Fonts loading via `<link>` for Playfair Display and Inter, as Shopify's font_picker may not include Playfair Display. If it does, use font_picker; if not, use direct Google Fonts links. The `font_picker` setting should still exist for fallback/override, but the theme.liquid should preconnect to Google Fonts.

#### 6. Section Groups

**`sections/header-group.json`**

```json
{
  "type": "header",
  "name": "Header group",
  "sections": {
    "announcement": { "type": "announcement-bar", "settings": {} },
    "header": { "type": "header", "settings": {} }
  },
  "order": ["announcement", "header"]
}
```

**`sections/footer-group.json`**

```json
{
  "type": "footer",
  "name": "Footer group",
  "sections": {
    "footer": { "type": "footer", "settings": {} }
  },
  "order": ["footer"]
}
```

#### 7. `layout/password.liquid`

Minimal password page layout for store password protection.

#### 8. `locales/en.default.json`

See [Translation Keys](#translation-keys) section below.

#### 9. `locales/en.default.schema.json`

See [Translation Keys](#translation-keys) section below.

**Phase 1 done when:** `shopify theme dev` serves a blank page with correct `<head>`, CSS variables render in DevTools, fonts load correctly, `validate_theme` passes with zero errors.

---

### Phase 2: Shared Snippets

Each snippet gets a `{% doc %}` header with parameter documentation. Snippets use raw `<style>` tags (lesson #6). Pre-assign translated strings before passing to `{% render %}` (lesson #5).

| Snippet | Parameters | Types | Required | Notes |
|---|---|---|---|---|
| `icon.liquid` | `name`: string | One of: cart, search, account, close, menu, arrow-right, play, instagram, check, minus, plus, star, star-filled, remove, chevron-right | Yes | New icons: play, instagram, remove |
| | `size`: number | Default: 24 | No | |
| | `class`: string | Additional CSS class | No | |
| `button.liquid` | `label`: string | Button text | Yes | |
| | `url`: string | If set, renders `<a>`; if unset, renders `<button>` | No | |
| | `variant`: string | One of: primary, outline, dark, green. Default: primary | No | **New:** primary=accent color, green=forest green (for add-to-cart) |
| | `type`: string | Button type attribute. Default: button | No | |
| | `full_width`: boolean | Default: false | No | |
| | `icon_after`: string | Icon name to render after label | No | **New:** For arrow icons on CTAs |
| `badge.liquid` | `type`: string | One of: new, sale, bestseller, featured, sold_out | Yes | **New:** `featured` type for collection cards |
| | `label`: string | Badge text | Yes | |
| `price.liquid` | `product`: product | Shopify product object | Yes | Renders green current price + strikethrough compare price |
| `star-rating.liquid` | `rating`: number | 0-5 value | Yes | Stars use accent color |
| | `count`: number | Review count to display | No | |
| `quantity-selector.liquid` | `value`: number | Current quantity. Default: 1 | No | |
| | `name`: string | Input name attribute | Yes | |
| | `line_item_key`: string | For cart AJAX updates | No | |
| `product-card.liquid` | `product`: product | Shopify product object | Yes | **Redesigned:** 4:5 aspect ratio, Quick View overlay, category label, Playfair title, green price |
| | `show_quick_view`: boolean | Default: true | No | **New:** Quick View hover button |
| `payment-icons.liquid` | (none) | Uses `shop.enabled_payment_types` directly | -- | Dark footer variant styling |
| `pagination.liquid` | `paginate`: paginate | Shopify paginate object | Yes | |
| `social-icons.liquid` | (none) | Reads from `settings.social_*` | -- | **New:** Renders social media icon links for footer |
| `breadcrumb.liquid` | `product`: product | Shopify product object | No | **New:** Breadcrumb navigation for PDP |
| | `collection`: collection | Shopify collection object | No | |

**Phase 2 done when:** Each snippet renders correctly when passed test parameters via `{% render %}` in a temporary test section. All `{% doc %}` headers present. `validate_theme` passes.

---

### Phase 3: Blocks

All blocks use `{% content_for 'blocks' %}` in their parent section (lesson #2) and `"type": "@theme"` in section schema (lesson #3).

#### collection-card block (`blocks/collection_card.liquid`)

Editorial-style collection card with image, dark gradient overlay, title, piece count, optional "Featured" tag. First card in grid spans 2 rows (handled by parent section CSS, not the block itself).

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `collection` | `collection` | (none) | `blocks.collection_card.settings.collection.label` |
| `image` | `image_picker` | (none) | `blocks.collection_card.settings.image.label` |
| `show_featured_tag` | `checkbox` | false | `blocks.collection_card.settings.show_featured_tag.label` |

Falls back to `collection.image` if custom image not set. Displays `collection.products_count` as "X pieces".

---

#### testimonial block (`blocks/testimonial.liquid`)

Card with star rating, italic Playfair quote, author avatar, name, and role.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `rating` | `range` | 5 (min: 1, max: 5, step: 1) | `blocks.testimonial.settings.rating.label` |
| `quote` | `textarea` | `"Add testimonial text"` | `blocks.testimonial.settings.quote.label` |
| `author_name` | `text` | `"Author"` | `blocks.testimonial.settings.author_name.label` |
| `author_role` | `text` | `""` | `blocks.testimonial.settings.author_role.label` |
| `author_image` | `image_picker` | (none) | `blocks.testimonial.settings.author_image.label` |

---

#### social-feed-item block (`blocks/social_feed_item.liquid`)

Square image with green overlay + Instagram icon on hover. Links to Instagram post.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `image` | `image_picker` | (none) | `blocks.social_feed_item.settings.image.label` |
| `url` | `url` | (none) | `blocks.social_feed_item.settings.url.label` |

---

#### marquee-item block (`blocks/marquee_item.liquid`)

Single text item in the scrolling marquee strip. Renders text + dot separator.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `text` | `text` | `"Marquee text"` | `blocks.marquee_item.settings.text.label` |

---

#### feature-check block (`blocks/feature_check.liquid`)

Checkmark + text line for product detail features list (e.g., "Free shipping", "Certificate of authenticity").

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `text` | `text` | `"Feature text"` | `blocks.feature_check.settings.text.label` |

---

#### stat block (`blocks/stat.liquid`)

Single stat item for editorial section (number + label, e.g., "200+ / Artists").

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `number` | `text` | `"0"` | `blocks.stat.settings.number.label` |
| `label` | `text` | `"Label"` | `blocks.stat.settings.label.label` |

---

#### heading block (`blocks/heading.liquid`)

Reusable heading block with italic accent word support.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `heading` | `text` | `"Heading"` | `blocks.heading.settings.heading.label` |
| `size` | `select` | `h2` | `blocks.heading.settings.size.label` |
| | | | Options: h1, h2, h3, h4 |

---

#### text block (`blocks/text.liquid`)

Reusable rich text block.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `text` | `richtext` | `"<p>Add your text here</p>"` | `blocks.text.settings.text.label` |

---

#### image block (`blocks/image.liquid`)

Reusable image block.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `image` | `image_picker` | (none) | `blocks.image.settings.image.label` |
| `url` | `url` | (none) | `blocks.image.settings.url.label` |
| `alt` | `text` | `""` | `blocks.image.settings.alt.label` |

---

**Phase 3 done when:** Blocks render inside a test section with default settings. Schema validates. `validate_theme` passes.

---

### Phase 4: Sections

#### announcement-bar.liquid

**New section (not in starter-minimal).**

Green background bar with scrolling text offers. Sits above header in header-group.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `text` | `text` | `"Free shipping on all orders over $99"` | `sections.announcement_bar.settings.text.label` |
| `highlight_text` | `text` | `"$99"` | `sections.announcement_bar.settings.highlight_text.label` |
| `link` | `url` | (none) | `sections.announcement_bar.settings.link.label` |
| `show_bar` | `checkbox` | true | `sections.announcement_bar.settings.show_bar.label` |

**Blocks:** None
**Presets:** `[{ "name": "Announcement bar" }]`

---

#### header.liquid

Dark glassmorphism sticky header (`rgba(26,26,24,0.95)` + `backdrop-filter: blur(12px)`). Serif logo with italic accent (`Art*Vault*`), uppercase nav links, pill-shaped cart button with dot indicator.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `logo_text` | `text` | `"ArtVault"` | `sections.header.settings.logo_text.label` |
| `logo_image` | `image_picker` | (none) | `sections.header.settings.logo_image.label` |
| `logo_width` | `range` | 140 (min: 80, max: 240, step: 10) | `sections.header.settings.logo_width.label` |
| `menu` | `link_list` | `main-menu` | `sections.header.settings.menu.label` |
| `show_search` | `checkbox` | true | `sections.header.settings.show_search.label` |
| `show_account` | `checkbox` | true | `sections.header.settings.show_account.label` |

**Blocks:** None
**Presets:** `[{ "name": "Header" }]`

**Architectural notes:**
- If `logo_image` is not set, renders text logo with Playfair Display where the second word is italic/accent colored
- Mobile: `<dialog>` for navigation drawer
- Cart button uses pill shape with green bg, dot indicator, and item count
- JS: mobile dialog toggle, `cart:updated` listener for badge count

---

#### hero-banner.liquid

Full-viewport hero with gradient overlay (green-to-dark), badge pill, huge serif heading with italic accent word, subtitle, two CTAs, and optional play button for video.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `image` | `image_picker` | (none) | `sections.hero_banner.settings.image.label` |
| `video_url` | `video_url` | (none) | `sections.hero_banner.settings.video_url.label` |
| `badge_text` | `text` | `""` | `sections.hero_banner.settings.badge_text.label` |
| `heading` | `text` | `"Where Art Meets *Life*"` | `sections.hero_banner.settings.heading.label` |
| `subtitle` | `textarea` | `"Discover handcrafted pieces from independent artists worldwide."` | `sections.hero_banner.settings.subtitle.label` |
| `button_label_1` | `text` | `"Explore collection"` | `sections.hero_banner.settings.button_label_1.label` |
| `button_url_1` | `url` | `/collections/all` | `sections.hero_banner.settings.button_url_1.label` |
| `button_label_2` | `text` | `"Watch story"` | `sections.hero_banner.settings.button_label_2.label` |
| `button_url_2` | `url` | (none) | `sections.hero_banner.settings.button_url_2.label` |
| `show_play_button` | `checkbox` | false | `sections.hero_banner.settings.show_play_button.label` |
| `overlay_opacity` | `range` | 75 (min: 0, max: 100, step: 5) | `sections.hero_banner.settings.overlay_opacity.label` |
| `height` | `select` | `full` | `sections.hero_banner.settings.height.label` |
| | | | Options: medium (600px), large (80vh), full (100vh) |

**Heading italic convention:** Text wrapped in `*asterisks*` in the heading setting renders as `<em>` with accent-light color. The Liquid template splits the heading string on `*` and wraps alternating segments.

**Blocks:** None
**Presets:** `[{ "name": "Hero banner" }]`

---

#### marquee.liquid

**New section (not in starter-minimal).**

Dark background scrolling text strip with dot separators. Infinite horizontal scroll animation via CSS `@keyframes`. Text is uppercase Playfair Display.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `speed` | `range` | 30 (min: 10, max: 60, step: 5) | `sections.marquee.settings.speed.label` |

**Blocks:** `marquee_item` (type: `@theme`, max: 12)
**Presets:**
```json
[{
  "name": "Marquee",
  "blocks": [
    { "type": "marquee_item", "settings": { "text": "Handcrafted" } },
    { "type": "marquee_item", "settings": { "text": "Sustainable" } },
    { "type": "marquee_item", "settings": { "text": "Artist-made" } },
    { "type": "marquee_item", "settings": { "text": "Limited edition" } },
    { "type": "marquee_item", "settings": { "text": "Ethically sourced" } },
    { "type": "marquee_item", "settings": { "text": "Free shipping" } }
  ]
}]
```

**CSS animation:** Two copies of the block list side-by-side, translating left infinitely. Pure CSS, no JavaScript.

---

#### collections-grid.liquid

**New section (not in starter-minimal). Replaces `category-grid.liquid`.**

Editorial 3-column grid where the first item spans 2 rows. Tight 8px gaps. Dark gradient overlays with title and piece count at bottom. Optional "Featured" tag.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `heading` | `text` | `"Curated *Collections*"` | `sections.collections_grid.settings.heading.label` |
| `subtitle` | `textarea` | `""` | `sections.collections_grid.settings.subtitle.label` |
| `button_label` | `text` | `"View all"` | `sections.collections_grid.settings.button_label.label` |
| `button_url` | `url` | `/collections` | `sections.collections_grid.settings.button_url.label` |

**Blocks:** `collection_card` (type: `@theme`, max: 8)
**Presets:** `[{ "name": "Collections grid" }]`

**CSS note:** `:first-child` of grid gets `grid-row: span 2` and loses the fixed aspect-ratio, filling the full height instead.

---

#### featured-products.liquid

Centered heading with italic accent, subtitle, category filter tabs (pills), 4-column product card grid. Uses `product-card` snippet.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `heading` | `text` | `"New *Arrivals*"` | `sections.featured_products.settings.heading.label` |
| `subtitle` | `text` | `"Fresh from the studios of our featured artists"` | `sections.featured_products.settings.subtitle.label` |
| `collection` | `collection` | (none) | `sections.featured_products.settings.collection.label` |
| `products_count` | `range` | 8 (min: 4, max: 16, step: 4) | `sections.featured_products.settings.products_count.label` |
| `show_collection_tabs` | `checkbox` | true | `sections.featured_products.settings.show_collection_tabs.label` |
| `columns_desktop` | `select` | `4` | `sections.featured_products.settings.columns_desktop.label` |
| | | | Options: 3, 4 |
| `show_quick_view` | `checkbox` | true | `sections.featured_products.settings.show_quick_view.label` |

**Blocks:** None
**Presets:** `[{ "name": "Featured products" }]`

**Architectural note -- collection tabs:** Tabs render as links to collection URLs (`/collections/paintings`, `/collections/prints`, etc.). The "All" tab links to the parent collection. This uses Shopify's native collection routing rather than client-side filtering. The tabs are populated from the sub-collections of the selected collection, or from a menu link_list if no collection is set.

---

#### editorial-split.liquid

**New section (not in starter-minimal).**

50/50 grid: image left, dark green content area right. Contains heading with italic accent, body text, CTA button, and stat counters.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `image` | `image_picker` | (none) | `sections.editorial_split.settings.image.label` |
| `heading` | `text` | `"Every Piece Tells a *Story*"` | `sections.editorial_split.settings.heading.label` |
| `text` | `textarea` | `""` | `sections.editorial_split.settings.text.label` |
| `button_label` | `text` | `"Meet our artists"` | `sections.editorial_split.settings.button_label.label` |
| `button_url` | `url` | (none) | `sections.editorial_split.settings.button_url.label` |
| `reverse_layout` | `checkbox` | false | `sections.editorial_split.settings.reverse_layout.label` |

**Blocks:** `stat` (type: `@theme`, max: 4)
**Presets:**
```json
[{
  "name": "Editorial split",
  "blocks": [
    { "type": "stat", "settings": { "number": "200+", "label": "Artists" } },
    { "type": "stat", "settings": { "number": "12k", "label": "Pieces sold" } },
    { "type": "stat", "settings": { "number": "48", "label": "Countries" } }
  ]
}]
```

---

#### social-feed.liquid

**New section (not in starter-minimal).**

Heading with Instagram handle, 6-column edge-to-edge image grid with green overlay + Instagram icon on hover.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `heading` | `text` | `"Follow the *Movement*"` | `sections.social_feed.settings.heading.label` |
| `subtitle` | `text` | `"See how our community styles their spaces"` | `sections.social_feed.settings.subtitle.label` |
| `instagram_handle` | `text` | `""` | `sections.social_feed.settings.instagram_handle.label` |
| `instagram_url` | `url` | (none) | `sections.social_feed.settings.instagram_url.label` |

**Blocks:** `social_feed_item` (type: `@theme`, max: 12)
**Presets:** `[{ "name": "Social feed" }]`

---

#### testimonials.liquid

**New section (not in starter-minimal).**

Centered heading, 3-column card grid. Each card has star rating, italic Playfair quote, author avatar + info.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `heading` | `text` | `"What People *Say*"` | `sections.testimonials.settings.heading.label` |

**Blocks:** `testimonial` (type: `@theme`, max: 9)
**Presets:**
```json
[{
  "name": "Testimonials",
  "blocks": [
    { "type": "testimonial", "settings": { "rating": 5, "quote": "The quality is outstanding. Every piece I've ordered has exceeded my expectations.", "author_name": "Sarah M.", "author_role": "Interior Designer" } },
    { "type": "testimonial", "settings": { "rating": 5, "quote": "Supporting independent artists while getting museum-quality pieces for my home? It's a win-win.", "author_name": "James K.", "author_role": "Art Collector" } },
    { "type": "testimonial", "settings": { "rating": 5, "quote": "The packaging was beautiful and the piece arrived perfectly. The artist even included a handwritten note.", "author_name": "Elena R.", "author_role": "Creative Director" } }
  ]
}]
```

---

#### newsletter.liquid

**Significantly different from starter-minimal's CTA banner.**

Green gradient overlay on background image. Centered serif heading with italic accent, subtitle, email input + subscribe button. Uses Shopify's customer form.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `image` | `image_picker` | (none) | `sections.newsletter.settings.image.label` |
| `heading` | `text` | `"Join the *Creative* Circle"` | `sections.newsletter.settings.heading.label` |
| `subtitle` | `textarea` | `"Get early access to new collections, artist stories, and exclusive offers."` | `sections.newsletter.settings.subtitle.label` |

**Blocks:** None
**Presets:** `[{ "name": "Newsletter" }]`

---

#### footer.liquid

Dark background footer. 4-column grid: brand (2fr) with logo, about text, social icons + 3 link columns (1fr each). Bottom bar with copyright and payment icons.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `about_text` | `textarea` | `""` | `sections.footer.settings.about_text.label` |
| `menu_1` | `link_list` | `footer-shop` | `sections.footer.settings.menu_1.label` |
| `menu_1_heading` | `text` | `"Shop"` | `sections.footer.settings.menu_1_heading.label` |
| `menu_2` | `link_list` | `footer-company` | `sections.footer.settings.menu_2.label` |
| `menu_2_heading` | `text` | `"Company"` | `sections.footer.settings.menu_2_heading.label` |
| `menu_3` | `link_list` | `footer-support` | `sections.footer.settings.menu_3.label` |
| `menu_3_heading` | `text` | `"Support"` | `sections.footer.settings.menu_3_heading.label` |
| `show_payment_icons` | `checkbox` | true | `sections.footer.settings.show_payment_icons.label` |

**Blocks:** None
**Presets:** `[{ "name": "Footer" }]`

**Differences from starter-minimal:** 3 menu columns instead of 2, social icons snippet, ArtVault serif logo in footer brand area.

---

#### product-detail.liquid

2-column layout: gallery (vertical thumbnails + main image) on left, product info on right. Breadcrumb, Playfair serif title, star rating, green price, variant selectors (size pills + frame/color swatches), quantity + green "Add to Cart" button, feature checklist.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `show_breadcrumb` | `checkbox` | true | `sections.product_detail.settings.show_breadcrumb.label` |
| `show_rating` | `checkbox` | true | `sections.product_detail.settings.show_rating.label` |
| `show_stock` | `checkbox` | true | `sections.product_detail.settings.show_stock.label` |
| `show_description` | `checkbox` | true | `sections.product_detail.settings.show_description.label` |

**Blocks:** `feature_check` (type: `@theme`, max: 8)
**Presets:**
```json
[{
  "name": "Product detail",
  "blocks": [
    { "type": "feature_check", "settings": { "text": "Free shipping on orders over $99" } },
    { "type": "feature_check", "settings": { "text": "Certificate of authenticity" } },
    { "type": "feature_check", "settings": { "text": "30-day returns" } },
    { "type": "feature_check", "settings": { "text": "Handcrafted by the artist" } }
  ]
}]
```

**Architectural notes:**
- Thumbnail gallery: vertical column of small images, clicking changes the main image. Pure JS (no libraries).
- Variant selectors use Shopify's native `product.variants` and `product.options`. Size renders as pill buttons. Color/frame renders as swatches (color circles).
- `{% form 'product' %}` for add-to-cart with AJAX submission
- Dispatches `cart:updated` CustomEvent after successful add

---

#### product-tabs.liquid

Uses `<details>`/`<summary>` for Description, Shipping & Returns, Reviews tabs.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `show_description` | `checkbox` | true | `sections.product_tabs.settings.show_description.label` |
| `show_shipping` | `checkbox` | true | `sections.product_tabs.settings.show_shipping.label` |
| `shipping_text` | `richtext` | `""` | `sections.product_tabs.settings.shipping_text.label` |
| `show_reviews` | `checkbox` | true | `sections.product_tabs.settings.show_reviews.label` |

**Blocks:** None
**Presets:** `[{ "name": "Product tabs" }]`

---

#### related-products.liquid

Uses Shopify product recommendations API (`/recommendations/products.json`). 4-column grid using `product-card` snippet.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `heading` | `text` | `"You may also like"` | `sections.related_products.settings.heading.label` |
| `products_count` | `range` | 4 (min: 2, max: 8, step: 1) | `sections.related_products.settings.products_count.label` |

**Blocks:** None
**Presets:** `[{ "name": "Related products" }]`

---

#### product-grid.liquid

Collection page product grid. Toolbar with count + sort dropdown, category filter tabs (collection tags as pills), paginated product grid. Uses `product-card` snippet.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `products_per_page` | `range` | 12 (min: 4, max: 24, step: 4) | `sections.product_grid.settings.products_per_page.label` |
| `columns_desktop` | `select` | `4` | `sections.product_grid.settings.columns_desktop.label` |
| | | | Options: 3, 4 |
| `show_sort` | `checkbox` | true | `sections.product_grid.settings.show_sort.label` |
| `show_collection_tabs` | `checkbox` | true | `sections.product_grid.settings.show_collection_tabs.label` |

**Blocks:** None
**Presets:** `[{ "name": "Product grid" }]`

---

#### cart-content.liquid

Two-column layout: left = line items (image, Playfair title, variant, quantity selector, price, remove link), right = sticky summary (Playfair heading, subtotal, shipping, tax, total, green Checkout button). AJAX updates via `/cart/change.js`.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `show_free_shipping_bar` | `checkbox` | true | `sections.cart_content.settings.show_free_shipping_bar.label` |
| `show_payment_icons` | `checkbox` | true | `sections.cart_content.settings.show_payment_icons.label` |

**Blocks:** None
**Presets:** `[{ "name": "Cart content" }]`

---

#### main-page.liquid

Renders `{{ page.content }}` for generic CMS pages.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `show_title` | `checkbox` | true | `sections.main_page.settings.show_title.label` |

**Blocks:** None
**Presets:** `[{ "name": "Page content" }]`

---

#### search-results.liquid

Search form + results grid using `search.results`. Uses `product-card` snippet.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `products_per_page` | `range` | 12 (min: 4, max: 24, step: 4) | `sections.search_results.settings.products_per_page.label` |

**Blocks:** None
**Presets:** `[{ "name": "Search results" }]`

---

#### not-found.liquid

404 page with heading, message, and CTA button.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `heading` | `text` | `"Page not found"` | `sections.not_found.settings.heading.label` |
| `message` | `textarea` | `"The page you're looking for doesn't exist."` | `sections.not_found.settings.message.label` |
| `button_label` | `text` | `"Back to home"` | `sections.not_found.settings.button_label.label` |
| `button_url` | `url` | `/` | `sections.not_found.settings.button_url.label` |

**Blocks:** None
**Presets:** `[{ "name": "404 page" }]`

---

#### collections-list.liquid

Grid of all collections with editorial card styling.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `heading` | `text` | `"All collections"` | `sections.collections_list.settings.heading.label` |

**Blocks:** None
**Presets:** `[{ "name": "Collections list" }]`

---

#### customer-login.liquid, customer-register.liquid, customer-account.liquid, customer-order.liquid

Standard customer pages. Same structure as starter-minimal but restyled with ArtVault typography and colors.

**Settings:** None
**Blocks:** None

---

**Phase 4 done when:** All sections render with default settings in the theme editor. Schema validates. `validate_theme` passes.

---

### Phase 5: JSON Templates

#### templates/index.json

```json
{
  "sections": {
    "hero": { "type": "hero-banner", "settings": {} },
    "marquee": { "type": "marquee", "settings": {} },
    "collections": {
      "type": "collections-grid",
      "settings": {},
      "blocks": {
        "col_1": { "type": "shopify://blocks/collection_card", "settings": { "show_featured_tag": true } },
        "col_2": { "type": "shopify://blocks/collection_card", "settings": {} },
        "col_3": { "type": "shopify://blocks/collection_card", "settings": {} },
        "col_4": { "type": "shopify://blocks/collection_card", "settings": {} },
        "col_5": { "type": "shopify://blocks/collection_card", "settings": {} }
      },
      "block_order": ["col_1", "col_2", "col_3", "col_4", "col_5"]
    },
    "featured": { "type": "featured-products", "settings": {} },
    "editorial": {
      "type": "editorial-split",
      "settings": {},
      "blocks": {
        "stat_1": { "type": "shopify://blocks/stat", "settings": { "number": "200+", "label": "Artists" } },
        "stat_2": { "type": "shopify://blocks/stat", "settings": { "number": "12k", "label": "Pieces sold" } },
        "stat_3": { "type": "shopify://blocks/stat", "settings": { "number": "48", "label": "Countries" } }
      },
      "block_order": ["stat_1", "stat_2", "stat_3"]
    },
    "social": { "type": "social-feed", "settings": {} },
    "testimonials": { "type": "testimonials", "settings": {} },
    "newsletter": { "type": "newsletter", "settings": {} }
  },
  "order": ["hero", "marquee", "collections", "featured", "editorial", "social", "testimonials", "newsletter"]
}
```

#### templates/collection.json

```json
{
  "sections": {
    "main": { "type": "product-grid", "settings": {} }
  },
  "order": ["main"]
}
```

#### templates/product.json

```json
{
  "sections": {
    "main": {
      "type": "product-detail",
      "settings": {},
      "blocks": {
        "feat_1": { "type": "shopify://blocks/feature_check", "settings": { "text": "Free shipping on orders over $99" } },
        "feat_2": { "type": "shopify://blocks/feature_check", "settings": { "text": "Certificate of authenticity" } },
        "feat_3": { "type": "shopify://blocks/feature_check", "settings": { "text": "30-day returns" } },
        "feat_4": { "type": "shopify://blocks/feature_check", "settings": { "text": "Handcrafted by the artist" } }
      },
      "block_order": ["feat_1", "feat_2", "feat_3", "feat_4"]
    },
    "tabs": { "type": "product-tabs", "settings": {} },
    "related": { "type": "related-products", "settings": {} }
  },
  "order": ["main", "tabs", "related"]
}
```

#### templates/cart.json

```json
{
  "sections": {
    "main": { "type": "cart-content", "settings": {} }
  },
  "order": ["main"]
}
```

#### templates/page.json

```json
{
  "sections": {
    "main": { "type": "main-page", "settings": {} }
  },
  "order": ["main"]
}
```

#### templates/search.json

```json
{
  "sections": {
    "main": { "type": "search-results", "settings": {} }
  },
  "order": ["main"]
}
```

#### templates/404.json

```json
{
  "sections": {
    "main": { "type": "not-found", "settings": {} }
  },
  "order": ["main"]
}
```

#### templates/list-collections.json

```json
{
  "sections": {
    "main": { "type": "collections-list", "settings": {} }
  },
  "order": ["main"]
}
```

#### templates/customers/login.json

```json
{
  "sections": {
    "main": { "type": "customer-login", "settings": {} }
  },
  "order": ["main"]
}
```

#### templates/customers/register.json

```json
{
  "sections": {
    "main": { "type": "customer-register", "settings": {} }
  },
  "order": ["main"]
}
```

#### templates/customers/account.json

```json
{
  "sections": {
    "main": { "type": "customer-account", "settings": {} }
  },
  "order": ["main"]
}
```

#### templates/customers/order.json

```json
{
  "sections": {
    "main": { "type": "customer-order", "settings": {} }
  },
  "order": ["main"]
}
```

#### templates/gift_card.liquid

Standard gift card template (must be Liquid, not JSON).

**Phase 5 done when:** All JSON templates parse without errors. `validate_theme` passes. `shopify theme dev` renders the home page with all sections.

---

### Phase 6: JavaScript (integrated into sections)

All vanilla JS, using `<script>` tags inside sections (lesson #1). No `{% javascript %}` tags.

| Section/Snippet | JS Behavior |
|---|---|
| **header.liquid** | Mobile `<dialog>` toggle, `cart:updated` listener for cart badge count |
| **hero-banner.liquid** | Optional: video play button opens `<dialog>` with embedded video |
| **marquee.liquid** | Pure CSS animation (no JS needed). Optional: pause on hover via `animation-play-state` |
| **featured-products.liquid** | Collection tab click navigates to collection URL (standard `<a>` links, no JS needed) |
| **product-card.liquid** | Quick View button dispatches `quickview:open` CustomEvent with product handle; AJAX add-to-cart from Quick View |
| **product-detail.liquid** | Thumbnail gallery click swaps main image, variant selector updates URL/price/availability, quantity +/- buttons, AJAX `/cart/add.js`, `cart:updated` dispatch |
| **product-grid.liquid** | Sort dropdown `onchange` navigates to URL with `?sort_by=` param |
| **cart-content.liquid** | AJAX `/cart/change.js` for quantity updates and removal, live total recalculation, `cart:updated` dispatch |
| **related-products.liquid** | Fetch from `/recommendations/products.json`, render cards client-side |

#### Cross-component communication pattern

Same as starter-minimal. All AJAX cart operations dispatch a `cart:updated` CustomEvent on `document` with the cart object as `detail`. The header listens for this event to update the cart badge count.

```javascript
// After any cart AJAX call:
document.dispatchEvent(new CustomEvent('cart:updated', { detail: cartData }));

// In header.liquid <script>:
document.addEventListener('cart:updated', (e) => {
  const badge = document.querySelector('[data-cart-count]');
  if (badge) badge.textContent = e.detail.item_count;
});
```

#### Quick View pattern

The Quick View button on product cards opens a native `<dialog>` element. This dialog is a single shared element in `layout/theme.liquid` (or the header section). When a Quick View button is clicked, JS fetches the product data from `/products/{handle}.js` and populates the dialog with product image, title, price, variant selector, and add-to-cart form.

**Phase 6 done when:** All interactive features work: mobile menu opens/closes, add-to-cart updates badge, thumbnail gallery switches images, variant selectors work, cart updates live, marquee scrolls. No console errors.

---

## Build Order (dependencies flow top-to-bottom)

1. Directory scaffolding (`themes/starter-bold/`)
2. `config/` (settings_schema.json, settings_data.json)
3. `locales/` (en.default.json, en.default.schema.json)
4. `assets/critical.css` + `snippets/css-variables.liquid`
5. `layout/theme.liquid` + `layout/password.liquid` + section groups
6. All snippets (icon, button, badge, price, star-rating, quantity-selector, product-card, payment-icons, pagination, social-icons, breadcrumb)
7. All blocks (collection_card, testimonial, social_feed_item, marquee_item, feature_check, stat, heading, text, image)
8. `sections/announcement-bar.liquid` + `sections/header.liquid` + `sections/footer.liquid`
9. Home page sections (hero-banner, marquee, collections-grid, featured-products, editorial-split, social-feed, testimonials, newsletter)
10. `templates/index.json` -- **first testable milestone**
11. `sections/product-grid.liquid` + `templates/collection.json`
12. Product sections (product-detail, product-tabs, related-products) + `templates/product.json`
13. Cart section + `templates/cart.json`
14. Remaining sections + templates (main-page, search-results, not-found, collections-list, customers)
15. `templates/gift_card.liquid`
16. Quick View dialog (in header or theme.liquid)
17. Full validation pass

---

## New vs Reused from starter-minimal

### Completely new sections (6):
- `announcement-bar.liquid` -- scrolling announcement bar
- `marquee.liquid` -- scrolling text strip with CSS animation
- `collections-grid.liquid` -- editorial 3-column collection grid (replaces simpler `category-grid`)
- `editorial-split.liquid` -- 50/50 image + content with stats
- `social-feed.liquid` -- Instagram-style image grid
- `testimonials.liquid` -- testimonial card grid

### Significantly redesigned sections (5):
- `header.liquid` -- glassmorphism dark header vs simple white header
- `hero-banner.liquid` -- full-viewport with gradient overlay, badge pill, play button
- `footer.liquid` -- 3 menu columns, social icons, payment badges in dark theme
- `newsletter.liquid` -- replaces `cta-banner`, uses image bg with overlay
- `product-detail.liquid` -- thumbnail gallery, breadcrumb, swatches, feature checks

### Structurally similar sections (8):
- `featured-products.liquid` -- same concept but with filter tabs and different card style
- `product-grid.liquid` -- same concept with added collection tabs
- `product-tabs.liquid` -- same structure, restyled
- `related-products.liquid` -- same structure, restyled
- `cart-content.liquid` -- same structure, restyled
- `main-page.liquid` -- identical structure
- `search-results.liquid` -- identical structure
- `not-found.liquid` -- identical structure
- `collections-list.liquid` -- identical structure
- Customer sections -- identical structure

### New blocks (5):
- `testimonial` -- quote card with author
- `social_feed_item` -- image with overlay
- `marquee_item` -- text item for marquee
- `feature_check` -- checkmark + text for PDP
- `stat` -- number + label for editorial section

### New snippets (2):
- `social-icons.liquid` -- social media icon links
- `breadcrumb.liquid` -- breadcrumb navigation

---

## React-to-Shopify Mapping

| React/Design Concept | Shopify Equivalent |
|---|---|
| Glassmorphism header | CSS `backdrop-filter: blur()` + `rgba()` background on sticky header section |
| Marquee animation | CSS `@keyframes` infinite scroll, duplicated content for seamless loop |
| Collection grid with span | CSS Grid `grid-row: span 2` on `:first-child` |
| Quick View overlay | `<dialog>` element + fetch from `/products/{handle}.js` |
| Category filter tabs | `<a>` links to collection URLs (native Shopify routing) |
| Instagram feed | Static images in blocks (no API integration -- merchants upload images) |
| Star ratings | SVG stars via `star-rating.liquid` snippet |
| Italic accent text | `*asterisk*` convention in settings, split and render as `<em>` in Liquid |
| Variant swatches | Shopify `product.options` + JS variant selection |
| All other React concepts | Same mapping as starter-minimal (see PLAN-0001) |

---

## Translation Keys

### locales/en.default.json (storefront)

```json
{
  "general": {
    "skip_to_content": "Skip to content",
    "cart": "Cart",
    "search": "Search",
    "account": "Account",
    "menu": "Menu",
    "close": "Close",
    "loading": "Loading"
  },
  "announcement": {
    "default_text": "Free shipping on all orders over {{ threshold }}"
  },
  "header": {
    "search_placeholder": "Search products...",
    "cart_count": "{{ count }} items"
  },
  "hero": {
    "default_heading": "Where Art Meets *Life*",
    "default_subtitle": "Discover handcrafted pieces from independent artists worldwide.",
    "explore_collection": "Explore collection",
    "watch_story": "Watch story"
  },
  "marquee": {
    "handcrafted": "Handcrafted",
    "sustainable": "Sustainable",
    "artist_made": "Artist-made",
    "limited_edition": "Limited edition",
    "ethically_sourced": "Ethically sourced",
    "free_shipping": "Free shipping"
  },
  "collections": {
    "heading": "Curated *Collections*",
    "view_all": "View all",
    "featured": "Featured",
    "pieces_count": "{{ count }} pieces",
    "all_heading": "All collections"
  },
  "products": {
    "add_to_cart": "Add to cart",
    "sold_out": "Sold out",
    "new": "New",
    "sale": "Sale",
    "bestseller": "Bestseller",
    "quick_view": "Quick view",
    "from_price": "From {{ price }}",
    "reviews_count": "{{ rating }} ({{ count }} reviews)",
    "sort_by": "Sort by",
    "sort_featured": "Featured",
    "sort_best_selling": "Best selling",
    "sort_price_asc": "Price, low to high",
    "sort_price_desc": "Price, high to low",
    "sort_newest": "Newest",
    "sort_title_asc": "A-Z",
    "sort_title_desc": "Z-A",
    "showing_count": "Showing {{ count }} products",
    "featured_heading": "New *Arrivals*",
    "featured_subtitle": "Fresh from the studios of our featured artists",
    "view_all": "View all",
    "you_may_also_like": "You may also like",
    "all": "All",
    "in_stock": "In stock",
    "low_stock": "Only {{ count }} left",
    "out_of_stock": "Out of stock",
    "description": "Description",
    "shipping_returns": "Shipping & returns",
    "reviews": "Reviews",
    "added_to_cart": "Added to cart",
    "size": "Size",
    "frame": "Frame",
    "color": "Color"
  },
  "editorial": {
    "default_heading": "Every Piece Tells a *Story*",
    "meet_artists": "Meet our artists"
  },
  "social": {
    "heading": "Follow the *Movement*",
    "subtitle": "See how our community styles their spaces"
  },
  "testimonials": {
    "heading": "What People *Say*"
  },
  "newsletter": {
    "heading": "Join the *Creative* Circle",
    "subtitle": "Get early access to new collections, artist stories, and exclusive offers.",
    "placeholder": "Your email address",
    "subscribe": "Subscribe",
    "success": "Thanks for subscribing!"
  },
  "cart": {
    "title": "Your *Cart*",
    "empty": "Your cart is empty",
    "continue_shopping": "Continue shopping",
    "subtotal": "Subtotal",
    "shipping": "Shipping",
    "shipping_free": "Free",
    "shipping_at_checkout": "Calculated at checkout",
    "tax": "Tax",
    "tax_at_checkout": "Calculated at checkout",
    "total": "Total",
    "order_summary": "Order summary",
    "checkout": "Checkout",
    "secure_checkout": "Secure checkout powered by Shopify",
    "free_shipping_notice": "Free shipping on orders over {{ threshold }}",
    "free_shipping_remaining": "Add {{ amount }} more for free shipping",
    "free_shipping_achieved": "You qualify for free shipping!",
    "remove": "Remove",
    "quantity": "Quantity"
  },
  "footer": {
    "about_heading": "About ArtVault",
    "copyright": "{{ year }} ArtVault. All rights reserved.",
    "shop": "Shop",
    "company": "Company",
    "support": "Support"
  },
  "search": {
    "title": "Search",
    "placeholder": "Search products...",
    "no_results": "No results for \"{{ query }}\"",
    "results_count": "{{ count }} results for \"{{ query }}\""
  },
  "not_found": {
    "title": "Page not found",
    "message": "The page you're looking for doesn't exist.",
    "back_to_home": "Back to home"
  },
  "customer": {
    "login_title": "Login",
    "register_title": "Create account",
    "account_title": "My account",
    "email": "Email",
    "password": "Password",
    "first_name": "First name",
    "last_name": "Last name",
    "submit_login": "Sign in",
    "submit_register": "Create account",
    "order_history": "Order history",
    "order_number": "Order {{ number }}",
    "order_date": "Date",
    "order_total": "Total",
    "order_status": "Status",
    "no_orders": "You haven't placed any orders yet."
  },
  "accessibility": {
    "increase_quantity": "Increase quantity",
    "decrease_quantity": "Decrease quantity",
    "open_menu": "Open menu",
    "close_menu": "Close menu",
    "star_rating": "{{ rating }} out of 5 stars",
    "open_cart": "Open cart",
    "remove_item": "Remove {{ title }} from cart",
    "play_video": "Play video",
    "previous_image": "Previous image",
    "next_image": "Next image",
    "social_link": "Follow us on {{ platform }}"
  }
}
```

### locales/en.default.schema.json (editor labels)

```json
{
  "settings": {
    "colors": {
      "name": "Colors",
      "color_primary": { "label": "Primary" },
      "color_primary_light": { "label": "Primary light" },
      "color_primary_dark": { "label": "Primary dark" },
      "color_primary_contrast": { "label": "Primary contrast" },
      "color_accent": { "label": "Accent" },
      "color_accent_light": { "label": "Accent light" },
      "color_accent_dark": { "label": "Accent dark" },
      "color_background": { "label": "Background" },
      "color_background_alt": { "label": "Background alternate" },
      "color_background_dark": { "label": "Background dark" },
      "color_background_warm": { "label": "Background warm" },
      "color_text": { "label": "Text" },
      "color_text_secondary": { "label": "Text secondary" },
      "color_text_muted": { "label": "Text muted" },
      "color_border": { "label": "Border" },
      "color_error": { "label": "Error / Sale" },
      "color_success": { "label": "Success" }
    },
    "typography": {
      "name": "Typography",
      "font_heading": { "label": "Heading font" },
      "font_body": { "label": "Body font" },
      "font_scale": { "label": "Font size scale" }
    },
    "layout": {
      "name": "Layout",
      "page_width": { "label": "Page width" },
      "border_radius": { "label": "Border radius" },
      "border_radius_large": { "label": "Large border radius" },
      "spacing_scale": { "label": "Spacing scale" }
    },
    "cart": {
      "name": "Cart",
      "free_shipping_threshold": { "label": "Free shipping threshold" },
      "show_free_shipping_bar": { "label": "Show free shipping progress" }
    },
    "social": {
      "name": "Social media",
      "social_instagram": { "label": "Instagram URL" },
      "social_twitter": { "label": "X (Twitter) URL" },
      "social_facebook": { "label": "Facebook URL" },
      "social_pinterest": { "label": "Pinterest URL" }
    }
  },
  "sections": {
    "announcement_bar": {
      "name": "Announcement bar",
      "settings": {
        "text": { "label": "Announcement text" },
        "highlight_text": { "label": "Highlighted text" },
        "link": { "label": "Link" },
        "show_bar": { "label": "Show announcement bar" }
      }
    },
    "header": {
      "name": "Header",
      "settings": {
        "logo_text": { "label": "Logo text" },
        "logo_image": { "label": "Logo image" },
        "logo_width": { "label": "Logo width" },
        "menu": { "label": "Navigation menu" },
        "show_search": { "label": "Show search" },
        "show_account": { "label": "Show account link" }
      }
    },
    "hero_banner": {
      "name": "Hero banner",
      "settings": {
        "image": { "label": "Background image" },
        "video_url": { "label": "Video URL" },
        "badge_text": { "label": "Badge text" },
        "heading": { "label": "Heading" },
        "subtitle": { "label": "Subtitle" },
        "button_label_1": { "label": "First button label" },
        "button_url_1": { "label": "First button URL" },
        "button_label_2": { "label": "Second button label" },
        "button_url_2": { "label": "Second button URL" },
        "show_play_button": { "label": "Show play button" },
        "overlay_opacity": { "label": "Overlay opacity" },
        "height": { "label": "Section height" }
      }
    },
    "marquee": {
      "name": "Marquee",
      "settings": {
        "speed": { "label": "Scroll speed (seconds)" }
      }
    },
    "collections_grid": {
      "name": "Collections grid",
      "settings": {
        "heading": { "label": "Heading" },
        "subtitle": { "label": "Subtitle" },
        "button_label": { "label": "Button label" },
        "button_url": { "label": "Button URL" }
      }
    },
    "featured_products": {
      "name": "Featured products",
      "settings": {
        "heading": { "label": "Heading" },
        "subtitle": { "label": "Subtitle" },
        "collection": { "label": "Collection" },
        "products_count": { "label": "Number of products" },
        "show_collection_tabs": { "label": "Show collection tabs" },
        "columns_desktop": { "label": "Desktop columns" },
        "show_quick_view": { "label": "Show quick view" }
      }
    },
    "editorial_split": {
      "name": "Editorial split",
      "settings": {
        "image": { "label": "Image" },
        "heading": { "label": "Heading" },
        "text": { "label": "Body text" },
        "button_label": { "label": "Button label" },
        "button_url": { "label": "Button URL" },
        "reverse_layout": { "label": "Reverse layout" }
      }
    },
    "social_feed": {
      "name": "Social feed",
      "settings": {
        "heading": { "label": "Heading" },
        "subtitle": { "label": "Subtitle" },
        "instagram_handle": { "label": "Instagram handle" },
        "instagram_url": { "label": "Instagram URL" }
      }
    },
    "testimonials": {
      "name": "Testimonials",
      "settings": {
        "heading": { "label": "Heading" }
      }
    },
    "newsletter": {
      "name": "Newsletter",
      "settings": {
        "image": { "label": "Background image" },
        "heading": { "label": "Heading" },
        "subtitle": { "label": "Subtitle" }
      }
    },
    "footer": {
      "name": "Footer",
      "settings": {
        "about_text": { "label": "About text" },
        "menu_1": { "label": "First menu" },
        "menu_1_heading": { "label": "First menu heading" },
        "menu_2": { "label": "Second menu" },
        "menu_2_heading": { "label": "Second menu heading" },
        "menu_3": { "label": "Third menu" },
        "menu_3_heading": { "label": "Third menu heading" },
        "show_payment_icons": { "label": "Show payment icons" }
      }
    },
    "product_detail": {
      "name": "Product detail",
      "settings": {
        "show_breadcrumb": { "label": "Show breadcrumb" },
        "show_rating": { "label": "Show rating" },
        "show_stock": { "label": "Show stock indicator" },
        "show_description": { "label": "Show description" }
      }
    },
    "product_tabs": {
      "name": "Product tabs",
      "settings": {
        "show_description": { "label": "Show description tab" },
        "show_shipping": { "label": "Show shipping tab" },
        "shipping_text": { "label": "Shipping & returns text" },
        "show_reviews": { "label": "Show reviews tab" }
      }
    },
    "related_products": {
      "name": "Related products",
      "settings": {
        "heading": { "label": "Heading" },
        "products_count": { "label": "Number of products" }
      }
    },
    "product_grid": {
      "name": "Product grid",
      "settings": {
        "products_per_page": { "label": "Products per page" },
        "columns_desktop": { "label": "Desktop columns" },
        "show_sort": { "label": "Show sort dropdown" },
        "show_collection_tabs": { "label": "Show collection filter tabs" }
      }
    },
    "cart_content": {
      "name": "Cart content",
      "settings": {
        "show_free_shipping_bar": { "label": "Show free shipping progress" },
        "show_payment_icons": { "label": "Show payment icons" }
      }
    },
    "main_page": {
      "name": "Page content",
      "settings": {
        "show_title": { "label": "Show page title" }
      }
    },
    "search_results": {
      "name": "Search results",
      "settings": {
        "products_per_page": { "label": "Products per page" }
      }
    },
    "not_found": {
      "name": "404 page",
      "settings": {
        "heading": { "label": "Heading" },
        "message": { "label": "Message" },
        "button_label": { "label": "Button label" },
        "button_url": { "label": "Button URL" }
      }
    },
    "collections_list": {
      "name": "Collections list",
      "settings": {
        "heading": { "label": "Heading" }
      }
    }
  },
  "blocks": {
    "collection_card": {
      "name": "Collection card",
      "settings": {
        "collection": { "label": "Collection" },
        "image": { "label": "Custom image" },
        "show_featured_tag": { "label": "Show featured tag" }
      }
    },
    "testimonial": {
      "name": "Testimonial",
      "settings": {
        "rating": { "label": "Star rating" },
        "quote": { "label": "Quote text" },
        "author_name": { "label": "Author name" },
        "author_role": { "label": "Author role" },
        "author_image": { "label": "Author image" }
      }
    },
    "social_feed_item": {
      "name": "Social feed image",
      "settings": {
        "image": { "label": "Image" },
        "url": { "label": "Link URL" }
      }
    },
    "marquee_item": {
      "name": "Marquee item",
      "settings": {
        "text": { "label": "Text" }
      }
    },
    "feature_check": {
      "name": "Feature",
      "settings": {
        "text": { "label": "Feature text" }
      }
    },
    "stat": {
      "name": "Stat",
      "settings": {
        "number": { "label": "Number" },
        "label": { "label": "Label" }
      }
    },
    "heading": {
      "name": "Heading",
      "settings": {
        "heading": { "label": "Heading" },
        "size": { "label": "Heading size" }
      }
    },
    "text": {
      "name": "Text",
      "settings": {
        "text": { "label": "Text" }
      }
    },
    "image": {
      "name": "Image",
      "settings": {
        "image": { "label": "Image" },
        "url": { "label": "Link URL" },
        "alt": { "label": "Alt text" }
      }
    }
  }
}
```

---

## Verification

- **After each phase:** Run `validate_theme` via Shopify Dev MCP on `themes/starter-bold/`. Fix all errors.
- **Lessons applied:** Verify `<script>` tags (not `{% javascript %}`), `{% content_for 'blocks' %}` in sections, `"type": "@theme"` in schema, `.btn` in critical.css, pre-assigned translation variables for `{% render %}`, raw `<style>` in snippets.
- **Translation coverage:** Grep for bare English text in `.liquid` files outside `{% schema %}` defaults.
- **Accessibility:** Every interactive element has an accessible name, images have alt text, `<dialog>` has focus management, star ratings have aria-label.
- **Responsive:** Mobile-first CSS with `min-width` breakpoints at 768px, 1024px, and 1200px.
- **Final test:** `shopify theme dev` on a dev store, navigate all pages, add to cart, mobile menu, sort, Quick View, marquee animation, verify theme editor customization.

---

## File Count Estimate

~60-65 files total.

| Category | Count | Most Complex |
|---|---|---|
| Config | 2 | settings_schema.json |
| Layout | 2 | theme.liquid |
| Locales | 2 | en.default.json |
| Assets | 1 | critical.css |
| Snippets | 12 | product-card.liquid |
| Blocks | 9 | collection_card.liquid |
| Sections | 20 | product-detail.liquid, header.liquid |
| Section groups | 2 | header-group.json |
| Templates | 13 | index.json |
| **Total** | **~63** | |

---

## Architectural Decisions

### AD-1: Italic accent text convention

Headings that contain `*asterisk-wrapped*` text render those segments as `<em>` with accent-light color. This is a convention baked into the Liquid templates: the heading string is split on `*` markers and alternating segments are wrapped in `<em>`. This avoids needing separate settings for "plain" and "accented" portions of headings.

### AD-2: Collection tabs use native routing

The category filter tabs on both the home page and collection page render as standard `<a>` links to collection URLs. This avoids client-side filtering complexity and works with Shopify's native collection system. The "All" tab links to `/collections/all` or the parent collection.

### AD-3: Quick View uses shared dialog

A single `<dialog id="quick-view">` element lives in theme.liquid (or the header section). Product cards dispatch a CustomEvent; a global script listens, fetches product JSON from `/products/{handle}.js`, and populates the dialog. This avoids duplicating dialog markup per card.

### AD-4: Social feed is merchant-curated, not API-driven

The Instagram/social feed section uses manually uploaded images in blocks rather than an Instagram API integration. This is more reliable (no API tokens to expire), gives merchants full control over which images appear, and avoids third-party dependencies.

### AD-5: Marquee is pure CSS

The marquee scroll animation uses CSS `@keyframes` with duplicated content (the block list is rendered twice) to create a seamless infinite loop. No JavaScript is needed. The animation pauses on hover via `animation-play-state: paused`.

### AD-6: Button classes in critical.css

Per lesson #4 from starter-minimal, all `.btn` variants (`.btn--primary`, `.btn--outline`, `.btn--dark`, `.btn--green`, `.btn--full`) are defined in `critical.css` rather than in component-level `<style>` tags. This prevents injection order issues where buttons render unstyled before component CSS loads.
