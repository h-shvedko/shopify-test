# PLAN-0001: BrewMaster Shopify Theme Implementation

> Status: APPROVED
> Created: 2026-03-16
> Source: Figma Make file `IrYnqvFhgUXJAmw10tmxoe`

## Context

The Figma Make file contains a complete React+Tailwind SPA for **"BrewMaster"** — a German-language homebrew equipment e-commerce store. We need to convert this design into the first Shopify Liquid theme (`starter-minimal`) under `themes/starter-minimal/`, following the project's established conventions from CLAUDE.md.

**Key challenge:** The source is React+Tailwind with client-side routing and state. The target is Shopify OS 2.0 Liquid with server-side rendering, JSON templates, `{% stylesheet %}` scoped CSS (no Tailwind), and Shopify's native cart API.

---

## Design Summary (from Figma)

- **Brand:** BrewMaster — homebrew equipment store
- **Color scheme:** Amber-600 (`#d97706`) primary, white/gray-50 alternating backgrounds, gray-900 footer
- **Pages:** Home, Shop (collection), Product Detail, Cart, Checkout (handled by Shopify natively)
- **Components:** Sticky header with mobile drawer, hero banner, 4-feature bar, category grid, product cards with ratings/badges, product tabs, cart with AJAX, dark 4-column footer
- **Products:** 12 products across 6 categories (Braukessel, Garung, Zutaten, Abfullung, Zubehor, Starter-Sets)
- **Locale:** German in the design, but English as default locale (per project convention), with translation keys for all text

---

## Design Tokens

| Token | Value | Usage |
|---|---|---|
| Breakpoint tablet | `768px` | `@media (min-width: 768px)` |
| Breakpoint desktop | `1024px` | `@media (min-width: 1024px)` |
| Container max-width | from `settings.page_width` | `.container` class |
| Section padding (desktop) | `60px` vertical | All section wrappers |
| Section padding (mobile) | `40px` vertical | All section wrappers |
| Grid gap (desktop) | `24px` | Product grids, feature grids |
| Grid gap (mobile) | `16px` | Product grids, feature grids |

---

## Implementation Approach

### Phase 0: Scaffold Directory Structure

Create `themes/starter-minimal/` with all required subdirectories:
`assets/`, `blocks/`, `config/`, `layout/`, `locales/`, `sections/`, `snippets/`, `templates/`, `templates/customers/`

**Done when:** All directories exist and are empty.

### Phase 1: Foundation (config, layout, assets, locales)

**Files to create:**

#### 1. `config/settings_schema.json`

Global theme settings organized in groups:

**Group: Colors**

| ID | Type | Default | Purpose |
|---|---|---|---|
| `color_primary` | `color` | `#d97706` | Primary accent (amber-600) |
| `color_primary_contrast` | `color` | `#ffffff` | Text on primary bg |
| `color_background` | `color` | `#ffffff` | Page background |
| `color_background_alt` | `color` | `#f9fafb` | Alternate section bg (gray-50) |
| `color_text` | `color` | `#111827` | Body text (gray-900) |
| `color_text_secondary` | `color` | `#6b7280` | Secondary text (gray-500) |
| `color_footer_bg` | `color` | `#111827` | Footer background |
| `color_footer_text` | `color` | `#d1d5db` | Footer text |
| `color_border` | `color` | `#e5e7eb` | Border color (gray-200) |
| `color_error` | `color` | `#dc2626` | Error state |
| `color_success` | `color` | `#16a34a` | Success state |

**Group: Typography**

| ID | Type | Default |
|---|---|---|
| `font_heading` | `font_picker` | `helvetica_n7` |
| `font_body` | `font_picker` | `helvetica_n4` |
| `font_scale` | `range` | 100 (min: 80, max: 130, step: 5) |

**Group: Layout**

| ID | Type | Default |
|---|---|---|
| `page_width` | `range` | 1280 (min: 1000, max: 1600, step: 40) |
| `border_radius` | `range` | 10 (min: 0, max: 24, step: 2) |
| `spacing_scale` | `range` | 100 (min: 80, max: 130, step: 10) |

**Group: Cart**

| ID | Type | Default |
|---|---|---|
| `free_shipping_threshold` | `number` | 99 |
| `show_free_shipping_bar` | `checkbox` | true |

#### 2. `config/settings_data.json`

Minimal default values pointing to schema defaults.

#### 3. `assets/critical.css`

CSS reset, `:root` variable fallbacks, `.container`, `.visually-hidden`, base typography. Keep under 5KB.

#### 4. `snippets/css-variables.liquid`

Renders a `<style>` tag with `:root` CSS custom properties mapped from `settings` object (e.g., `--color-primary: {{ settings.color_primary }};`). Uses a raw `<style>` element since snippets cannot use `{% stylesheet %}`.

#### 5. `layout/theme.liquid`

Master layout: `<html>` with locale, `<head>` (meta, critical.css, css-variables, `content_for_header`), `<body>` with skip-to-content link, `{% sections 'header-group' %}`, `<main>{{ content_for_layout }}</main>`, `{% sections 'footer-group' %}`

#### 6. Section Groups

**`sections/header-group.json`**

```json
{
  "type": "header",
  "name": "Header group",
  "sections": {
    "header": { "type": "header", "settings": {} }
  },
  "order": ["header"]
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

#### 7. `locales/en.default.json`

See [Translation Keys](#translation-keys) section below.

#### 8. `locales/en.default.schema.json`

See [Translation Keys](#translation-keys) section below.

**Phase 1 done when:** `shopify theme dev` serves a blank page with correct `<head>`, CSS variables render in DevTools, `validate_theme` passes with zero errors.

### Phase 2: Shared Snippets

Each snippet gets a `{% doc %}` header with parameter documentation.

| Snippet | Parameters | Types | Required |
|---|---|---|---|
| `icon.liquid` | `name`: string | One of: cart, search, truck, shield, star, check, minus, plus, menu, close, arrow-left, phone, mail, map-pin, chevron-right, shopping-bag | Yes |
| | `size`: number | Default: 24 | No |
| | `class`: string | Additional CSS class | No |
| `button.liquid` | `label`: string | Button text | Yes |
| | `url`: string | If set, renders `<a>`; if unset, renders `<button>` | No |
| | `variant`: string | One of: primary, secondary, outline. Default: primary | No |
| | `type`: string | Button type attribute. Default: button | No |
| | `full_width`: boolean | Default: false | No |
| `badge.liquid` | `type`: string | One of: sold_out, bestseller, default | Yes |
| | `label`: string | Badge text | Yes |
| `price.liquid` | `product`: product | Shopify product object | Yes |
| `star-rating.liquid` | `rating`: number | 0-5 value | Yes |
| | `count`: number | Review count to display | No |
| `quantity-selector.liquid` | `value`: number | Current quantity. Default: 1 | No |
| | `name`: string | Input name attribute | Yes |
| | `line_item_key`: string | For cart AJAX updates | No |
| `product-card.liquid` | `product`: product | Shopify product object | Yes |
| | `show_description`: boolean | Default: false | No |
| | `show_add_to_cart`: boolean | Default: true | No |
| `payment-icons.liquid` | (none) | Uses `shop.enabled_payment_types` directly | — |
| `pagination.liquid` | `paginate`: paginate | Shopify paginate object | Yes |

**Phase 2 done when:** Each snippet renders correctly when passed test parameters via `{% render %}` in a temporary test section. All `{% doc %}` headers present. `validate_theme` passes.

### Phase 3: Blocks

#### feature-item block

- **Type:** `feature_item`
- **Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `icon` | `select` | `truck` | `blocks.feature_item.settings.icon.label` |
| | | | Options: truck, shield, star, check |
| `heading` | `text` | `"Feature"` | `blocks.feature_item.settings.heading.label` |
| `description` | `text` | `"Description"` | `blocks.feature_item.settings.description.label` |

#### category-card block

- **Type:** `category_card`
- **Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `collection` | `collection` | (none) | `blocks.category_card.settings.collection.label` |
| `image` | `image_picker` | (none) | `blocks.category_card.settings.image.label` |
| | | | Falls back to `collection.image` if not set |

#### text block

- **Type:** `text`
- **Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `text` | `richtext` | `"<p>Add your text here</p>"` | `blocks.text.settings.text.label` |

#### heading block

- **Type:** `heading`
- **Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `heading` | `text` | `"Heading"` | `blocks.heading.settings.heading.label` |
| `size` | `select` | `h2` | `blocks.heading.settings.size.label` |
| | | | Options: h1, h2, h3, h4 |

#### image block

- **Type:** `image`
- **Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `image` | `image_picker` | (none) | `blocks.image.settings.image.label` |
| `url` | `url` | (none) | `blocks.image.settings.url.label` |
| `alt` | `text` | `""` | `blocks.image.settings.alt.label` |

**Phase 3 done when:** Blocks render inside a test section with default settings. Schema validates. `validate_theme` passes.

### Phase 4: Sections

#### header.liquid

Maps from: Header.tsx

Sticky header with logo, navigation, search, cart badge, and mobile `<dialog>` menu.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `logo` | `image_picker` | (none) | `sections.header.settings.logo.label` |
| `logo_width` | `range` | 120 (min: 60, max: 200, step: 10) | `sections.header.settings.logo_width.label` |
| `menu` | `link_list` | `main-menu` | `sections.header.settings.menu.label` |

**Blocks:** None
**Presets:** `[{ "name": "Header" }]`

---

#### footer.liquid

Maps from: Footer.tsx

4-column dark footer: about text, 2 menus, contact info. Copyright with dynamic year.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `about_text` | `textarea` | `""` | `sections.footer.settings.about_text.label` |
| `menu_1` | `link_list` | `footer-1` | `sections.footer.settings.menu_1.label` |
| `menu_1_heading` | `text` | `"Customer service"` | `sections.footer.settings.menu_1_heading.label` |
| `menu_2` | `link_list` | `footer-2` | `sections.footer.settings.menu_2.label` |
| `menu_2_heading` | `text` | `"Information"` | `sections.footer.settings.menu_2_heading.label` |
| `show_newsletter` | `checkbox` | true | `sections.footer.settings.show_newsletter.label` |
| `phone` | `text` | `""` | `sections.footer.settings.phone.label` |
| `email` | `text` | `""` | `sections.footer.settings.email.label` |
| `address` | `textarea` | `""` | `sections.footer.settings.address.label` |

**Blocks:** None
**Presets:** `[{ "name": "Footer" }]`

---

#### hero-banner.liquid

Maps from: Home hero

Full-width background image with overlay, heading, subtitle, and 2 CTA buttons.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `image` | `image_picker` | (none) | `sections.hero_banner.settings.image.label` |
| `heading` | `text` | `"Brew your perfect beer"` | `sections.hero_banner.settings.heading.label` |
| `subtitle` | `textarea` | `"Premium homebrew equipment for every brewer"` | `sections.hero_banner.settings.subtitle.label` |
| `button_label_1` | `text` | `"Shop now"` | `sections.hero_banner.settings.button_label_1.label` |
| `button_url_1` | `url` | `/collections/all` | `sections.hero_banner.settings.button_url_1.label` |
| `button_label_2` | `text` | `"Learn more"` | `sections.hero_banner.settings.button_label_2.label` |
| `button_url_2` | `url` | (none) | `sections.hero_banner.settings.button_url_2.label` |
| `height` | `select` | `large` | `sections.hero_banner.settings.height.label` |
| | | | Options: small (400px), medium (500px), large (600px) |
| `overlay_opacity` | `range` | 40 (min: 0, max: 100, step: 10) | `sections.hero_banner.settings.overlay_opacity.label` |

**Blocks:** None
**Presets:** `[{ "name": "Hero banner" }]`

---

#### features-bar.liquid

Maps from: Home features

Gray-50 background, 4-column grid of `feature-item` blocks.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `heading` | `text` | `""` | `sections.features_bar.settings.heading.label` |

**Blocks:** `feature_item` (max: 8)
**Presets:**
```json
[{
  "name": "Features bar",
  "blocks": [
    { "type": "feature_item", "settings": { "icon": "truck", "heading": "Free shipping", "description": "On orders over $99" } },
    { "type": "feature_item", "settings": { "icon": "shield", "heading": "Secure payment", "description": "100% secure checkout" } },
    { "type": "feature_item", "settings": { "icon": "star", "heading": "Best quality", "description": "Premium materials" } },
    { "type": "feature_item", "settings": { "icon": "check", "heading": "Expert support", "description": "Brewing advice included" } }
  ]
}]
```

---

#### category-grid.liquid

Maps from: Home categories

Heading + grid of collection images with dark overlay text.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `heading` | `text` | `"Shop by category"` | `sections.category_grid.settings.heading.label` |

**Blocks:** `category_card` (max: 12)
**Presets:** `[{ "name": "Category grid" }]`

---

#### featured-products.liquid

Maps from: Home featured products

Heading + "View all" link + 4-column product grid. Uses `product-card` snippet.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `heading` | `text` | `"Featured products"` | `sections.featured_products.settings.heading.label` |
| `collection` | `collection` | (none) | `sections.featured_products.settings.collection.label` |
| `products_count` | `range` | 4 (min: 2, max: 12, step: 1) | `sections.featured_products.settings.products_count.label` |
| `show_view_all` | `checkbox` | true | `sections.featured_products.settings.show_view_all.label` |

**Blocks:** None
**Presets:** `[{ "name": "Featured products" }]`

---

#### cta-banner.liquid

Maps from: Home CTA

Full-width amber background with heading, subtitle, and button.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `heading` | `text` | `"Ready to start brewing?"` | `sections.cta_banner.settings.heading.label` |
| `subtitle` | `textarea` | `""` | `sections.cta_banner.settings.subtitle.label` |
| `button_label` | `text` | `"Shop now"` | `sections.cta_banner.settings.button_label.label` |
| `button_url` | `url` | `/collections/all` | `sections.cta_banner.settings.button_url.label` |

**Blocks:** None
**Presets:** `[{ "name": "CTA banner" }]`

---

#### product-grid.liquid

Maps from: Shop page

Toolbar (count + sort dropdown) + paginated product grid. Sort via Shopify native `?sort_by=`. Tag filtering. Uses `product-card` snippet.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `products_per_page` | `range` | 12 (min: 4, max: 24, step: 4) | `sections.product_grid.settings.products_per_page.label` |
| `columns_desktop` | `select` | `4` | `sections.product_grid.settings.columns_desktop.label` |
| | | | Options: 3, 4 |
| `show_sort` | `checkbox` | true | `sections.product_grid.settings.show_sort.label` |

**Blocks:** None
**Presets:** `[{ "name": "Product grid" }]`

---

#### product-detail.liquid

Maps from: ProductDetail.tsx

2-column layout: image on left, info on right (badge, title, rating, price, description, features, quantity selector, stock indicator, `{% form 'product' %}` add-to-cart). AJAX JavaScript for add-to-cart.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `show_rating` | `checkbox` | true | `sections.product_detail.settings.show_rating.label` |
| `show_stock` | `checkbox` | true | `sections.product_detail.settings.show_stock.label` |
| `show_description` | `checkbox` | true | `sections.product_detail.settings.show_description.label` |

**Blocks:** None
**Presets:** `[{ "name": "Product detail" }]`

Note: Product data comes from the `product` object in the template context, not from settings.

---

#### product-tabs.liquid

Maps from: Product tabs

Uses `<details>`/`<summary>` elements for Description, Shipping & Returns, Reviews.

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

Maps from: Related products section

Uses Shopify product recommendations API (`/recommendations/products.json`).

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `heading` | `text` | `"You may also like"` | `sections.related_products.settings.heading.label` |
| `products_count` | `range` | 4 (min: 2, max: 8, step: 1) | `sections.related_products.settings.products_count.label` |

**Blocks:** None
**Presets:** `[{ "name": "Related products" }]`

---

#### cart-content.liquid

Maps from: Cart.tsx (cart-items + cart-summary combined)

Two-column layout: left = line items (image, name, quantity-selector, price, remove), right = summary (subtotal, shipping estimate, tax, total, free shipping bar, checkout button, payment icons). Single section avoids cross-section layout issues. AJAX updates via `/cart/change.js`.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `show_free_shipping_bar` | `checkbox` | true | `sections.cart_content.settings.show_free_shipping_bar.label` |
| `show_payment_icons` | `checkbox` | true | `sections.cart_content.settings.show_payment_icons.label` |

**Blocks:** None
**Presets:** `[{ "name": "Cart content" }]`

---

#### main-page.liquid

Maps from: N/A

Renders `{{ page.content }}` for generic CMS pages.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `show_title` | `checkbox` | true | `sections.main_page.settings.show_title.label` |

**Blocks:** None
**Presets:** `[{ "name": "Page content" }]`

---

#### search-results.liquid

Maps from: N/A

Search form + results grid using `search.results`. Uses `product-card` snippet for product results. Displays articles and pages as simple links.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `products_per_page` | `range` | 12 (min: 4, max: 24, step: 4) | `sections.search_results.settings.products_per_page.label` |

**Blocks:** None
**Presets:** `[{ "name": "Search results" }]`

---

#### not-found.liquid

Maps from: N/A

404 page with heading, message, and link back to home.

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

Maps from: N/A

Grid of all collections with image and title. Uses `{{ collections }}` global object.

**Settings:**

| ID | Type | Default | Label key |
|---|---|---|---|
| `heading` | `text` | `"All collections"` | `sections.collections_list.settings.heading.label` |

**Blocks:** None
**Presets:** `[{ "name": "Collections list" }]`

---

#### customer-login.liquid

Maps from: N/A

Login form using `{% form 'customer_login' %}`.

**Settings:** None
**Blocks:** None

---

#### customer-register.liquid

Maps from: N/A

Register form using `{% form 'create_customer' %}`.

**Settings:** None
**Blocks:** None

---

#### customer-account.liquid

Maps from: N/A

Order history list using `customer.orders`.

**Settings:** None
**Blocks:** None

---

#### customer-order.liquid

Maps from: N/A

Single order detail using `order` object.

**Settings:** None
**Blocks:** None

---

**Architectural decision — Checkout:** Shopify handles checkout natively. The React checkout page does NOT need to be recreated. The cart links to `/checkout`.

**Architectural decision — Collection filtering:** The React sidebar category filter maps to Shopify's collection architecture. Each category = a Shopify collection. `/collections/all` = "all products". No client-side filter state needed.

**Architectural decision — Cart layout:** Cart items and cart summary are combined into a single `cart-content.liquid` section to enable a two-column CSS grid layout. Two separate sections would stack vertically.

**Phase 4 done when:** All sections render with default settings in the theme editor. Schema validates. `validate_theme` passes.

### Phase 5: JSON Templates

#### templates/index.json

```json
{
  "sections": {
    "hero": { "type": "hero-banner", "settings": {} },
    "features": {
      "type": "features-bar",
      "blocks": {
        "feature_1": { "type": "feature_item", "settings": { "icon": "truck", "heading": "Free shipping", "description": "On orders over $99" } },
        "feature_2": { "type": "feature_item", "settings": { "icon": "shield", "heading": "Secure payment", "description": "100% secure checkout" } },
        "feature_3": { "type": "feature_item", "settings": { "icon": "star", "heading": "Best quality", "description": "Premium materials" } },
        "feature_4": { "type": "feature_item", "settings": { "icon": "check", "heading": "Expert support", "description": "Brewing advice included" } }
      },
      "block_order": ["feature_1", "feature_2", "feature_3", "feature_4"]
    },
    "categories": { "type": "category-grid", "settings": {} },
    "featured": { "type": "featured-products", "settings": {} },
    "cta": { "type": "cta-banner", "settings": {} }
  },
  "order": ["hero", "features", "categories", "featured", "cta"]
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
    "main": { "type": "product-detail", "settings": {} },
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

**Phase 5 done when:** All JSON templates parse without errors. `validate_theme` passes. `shopify theme dev` renders the home page with all sections.

### Phase 6: JavaScript (integrated into sections)

All vanilla JS, one `{% javascript %}` tag per section/snippet:

- **Header:** Mobile `<dialog>` toggle, cart badge update listener
- **Product detail:** Quantity +/- buttons, AJAX `/cart/add.js`, cart badge update, toast notification
- **Product card (snippet):** AJAX add-to-cart on button click
- **Product grid:** Sort dropdown `onchange` → URL navigation
- **Cart content:** AJAX `/cart/change.js` for quantity updates and removal, live total recalculation
- **Related products:** Fetch from `/recommendations/products.json`

#### Cross-component communication pattern

All AJAX cart operations dispatch a `cart:updated` CustomEvent on `document` with the cart object as `detail`. The header section listens for this event to update the cart badge count.

```javascript
// After any cart AJAX call:
document.dispatchEvent(new CustomEvent('cart:updated', { detail: cartData }));

// In header.liquid {% javascript %}:
document.addEventListener('cart:updated', (e) => {
  document.querySelector('[data-cart-count]').textContent = e.detail.item_count;
});
```

**Phase 6 done when:** All interactive features work: mobile menu opens/closes, add-to-cart updates badge, quantity selector works, sort changes URL, cart updates live. No console errors.

---

## Build Order (dependencies flow top-to-bottom)

1. Directory scaffolding
2. `config/` (settings_schema.json, settings_data.json)
3. `locales/` (en.default.json, en.default.schema.json)
4. `assets/critical.css` + `snippets/css-variables.liquid`
5. `layout/theme.liquid` + section groups
6. All other snippets (icon, button, badge, price, star-rating, quantity-selector, product-card, payment-icons, pagination)
7. All blocks
8. `sections/header.liquid` + `sections/footer.liquid`
9. Home page sections (hero, features, categories, featured-products, cta)
10. `templates/index.json` — **first testable milestone**
11. `sections/product-grid.liquid` + `templates/collection.json`
12. Product sections (detail, tabs, related) + `templates/product.json`
13. Cart section + `templates/cart.json`
14. Remaining sections + templates (page, search, 404, list-collections, customers)
15. Full validation pass

---

## React-to-Shopify Mapping

| React Concept | Shopify Equivalent |
|---|---|
| `<App>` / `RootLayout` | `layout/theme.liquid` |
| React Router routes | JSON templates (`index.json`, `collection.json`, `product.json`) |
| React component (e.g., `ProductCard.tsx`) | Snippet (`snippets/product-card.liquid`) or Block |
| Page-level component (e.g., `HomePage`) | JSON template composing sections |
| Tailwind utility classes | Scoped CSS in `{% stylesheet %}` tags |
| `useState` / `useContext` (CartContext) | Shopify `cart` object + AJAX cart API + vanilla JS |
| `react-router` `<Link>` | `<a href="{{ url }}">` |
| Conditional rendering | `{% if condition %}...{% endif %}` |
| `.map()` over arrays | `{% for item in array %}...{% endfor %}` |
| Props drilling | `{% render 'snippet', param: value %}` |
| Lucide React icons | Inline SVGs in `snippets/icon.liquid` |
| Tailwind responsive (`sm:`, `md:`, `lg:`) | CSS `@media (min-width: ...)` |
| Sheet/Dialog (radix-ui) | Native `<dialog>` element |
| Toast notifications | CSS + JS positioned notifications |
| CartContext state | `cart:updated` CustomEvent on `document` |

---

## Translation Keys

### locales/en.default.json (storefront)

```json
{
  "general": {
    "skip_to_content": "Skip to content",
    "cart": "Cart",
    "search": "Search",
    "menu": "Menu",
    "close": "Close",
    "loading": "Loading"
  },
  "header": {
    "search_placeholder": "Search products...",
    "cart_count": "{{ count }} items"
  },
  "hero": {
    "default_heading": "Brew your perfect beer",
    "default_subtitle": "Premium homebrew equipment for every brewer",
    "shop_now": "Shop now",
    "learn_more": "Learn more"
  },
  "features": {
    "free_shipping": "Free shipping",
    "free_shipping_description": "On orders over {{ threshold }}",
    "secure_payment": "Secure payment",
    "secure_payment_description": "100% secure checkout",
    "best_quality": "Best quality",
    "best_quality_description": "Premium materials",
    "expert_support": "Expert support",
    "expert_support_description": "Brewing advice included"
  },
  "categories": {
    "heading": "Shop by category",
    "view_all": "View all"
  },
  "products": {
    "add_to_cart": "Add to cart",
    "sold_out": "Sold out",
    "from_price": "From {{ price }}",
    "reviews_count": "{{ count }} reviews",
    "sort_by": "Sort by",
    "sort_featured": "Featured",
    "sort_best_selling": "Best selling",
    "sort_price_asc": "Price, low to high",
    "sort_price_desc": "Price, high to low",
    "sort_newest": "Newest",
    "sort_title_asc": "A-Z",
    "sort_title_desc": "Z-A",
    "showing_count": "Showing {{ count }} products",
    "featured_heading": "Featured products",
    "view_all": "View all",
    "you_may_also_like": "You may also like",
    "in_stock": "In stock",
    "low_stock": "Only {{ count }} left",
    "out_of_stock": "Out of stock",
    "description": "Description",
    "shipping_returns": "Shipping & returns",
    "reviews": "Reviews",
    "added_to_cart": "Added to cart"
  },
  "cart": {
    "title": "Shopping cart",
    "empty": "Your cart is empty",
    "continue_shopping": "Continue shopping",
    "subtotal": "Subtotal",
    "shipping": "Shipping",
    "shipping_at_checkout": "Calculated at checkout",
    "tax": "Tax",
    "tax_included": "Tax included",
    "total": "Total",
    "checkout": "Checkout",
    "free_shipping_notice": "Free shipping on orders over {{ threshold }}",
    "free_shipping_remaining": "Add {{ amount }} more for free shipping",
    "free_shipping_achieved": "You qualify for free shipping!",
    "remove": "Remove",
    "quantity": "Quantity"
  },
  "footer": {
    "about_heading": "About BrewMaster",
    "customer_service": "Customer service",
    "information": "Information",
    "contact": "Contact",
    "copyright": "© {{ year }} BrewMaster. All rights reserved.",
    "newsletter_heading": "Subscribe to our newsletter",
    "newsletter_placeholder": "Your email address",
    "subscribe": "Subscribe"
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
  "collections": {
    "heading": "All collections"
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
    "remove_item": "Remove {{ title }} from cart"
  }
}
```

### locales/en.default.schema.json (editor labels)

```json
{
  "sections": {
    "header": {
      "name": "Header",
      "settings": {
        "logo": { "label": "Logo" },
        "logo_width": { "label": "Logo width" },
        "menu": { "label": "Navigation menu" }
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
        "show_newsletter": { "label": "Show newsletter signup" },
        "phone": { "label": "Phone number" },
        "email": { "label": "Email address" },
        "address": { "label": "Address" }
      }
    },
    "hero_banner": {
      "name": "Hero banner",
      "settings": {
        "image": { "label": "Background image" },
        "heading": { "label": "Heading" },
        "subtitle": { "label": "Subtitle" },
        "button_label_1": { "label": "First button label" },
        "button_url_1": { "label": "First button URL" },
        "button_label_2": { "label": "Second button label" },
        "button_url_2": { "label": "Second button URL" },
        "height": { "label": "Section height" },
        "overlay_opacity": { "label": "Overlay opacity" }
      }
    },
    "features_bar": {
      "name": "Features bar",
      "settings": {
        "heading": { "label": "Heading" }
      }
    },
    "category_grid": {
      "name": "Category grid",
      "settings": {
        "heading": { "label": "Heading" }
      }
    },
    "featured_products": {
      "name": "Featured products",
      "settings": {
        "heading": { "label": "Heading" },
        "collection": { "label": "Collection" },
        "products_count": { "label": "Number of products" },
        "show_view_all": { "label": "Show 'View all' link" }
      }
    },
    "cta_banner": {
      "name": "CTA banner",
      "settings": {
        "heading": { "label": "Heading" },
        "subtitle": { "label": "Subtitle" },
        "button_label": { "label": "Button label" },
        "button_url": { "label": "Button URL" }
      }
    },
    "product_grid": {
      "name": "Product grid",
      "settings": {
        "products_per_page": { "label": "Products per page" },
        "columns_desktop": { "label": "Desktop columns" },
        "show_sort": { "label": "Show sort dropdown" }
      }
    },
    "product_detail": {
      "name": "Product detail",
      "settings": {
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
    "cart_content": {
      "name": "Cart content",
      "settings": {
        "show_free_shipping_bar": { "label": "Show free shipping progress bar" },
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
    "feature_item": {
      "name": "Feature item",
      "settings": {
        "icon": { "label": "Icon" },
        "heading": { "label": "Heading" },
        "description": { "label": "Description" }
      }
    },
    "category_card": {
      "name": "Category card",
      "settings": {
        "collection": { "label": "Collection" },
        "image": { "label": "Custom image" }
      }
    },
    "text": {
      "name": "Text",
      "settings": {
        "text": { "label": "Text" }
      }
    },
    "heading": {
      "name": "Heading",
      "settings": {
        "heading": { "label": "Heading" },
        "size": { "label": "Heading size" }
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

- **After each phase:** Run `validate_theme` via Shopify Dev MCP on `themes/starter-minimal/`. Fix all errors.
- **Translation coverage:** Grep for bare English text in `.liquid` files outside `{% schema %}` defaults.
- **Accessibility:** Every interactive element has an accessible name, images have alt text, `<dialog>` has focus management.
- **Responsive:** Mobile-first CSS with `min-width` breakpoints at 768px and 1024px.
- **Final test:** `shopify theme dev` on a dev store, navigate all pages, add to cart, mobile menu, sort, verify theme editor customization.

---

## File Count Estimate

~45-50 files total. Most complex: `product-detail.liquid`, `header.liquid`, `cart-content.liquid`, `product-card.liquid`.
