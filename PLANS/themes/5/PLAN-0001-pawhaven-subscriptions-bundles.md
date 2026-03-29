# PLAN-0001: PawHaven Subscriptions and Bundles

**Theme:** PawHaven (starter-pets)
**Date:** 2026-03-28
**Status:** Draft
**ADR reference:** ADR-0002, Project 12 — Subscriptions go to BrewMaster, Bundles go to PawHaven. This plan covers both features in PawHaven to create a cohesive pet-supplies experience.

---

## Context

PawHaven is a 52-file Shopify OS 2.0 theme targeting pet supply merchants. The theme uses emerald (#059669) + warm sand (#F5F0E8), Nunito/Quicksand fonts, 12px border radius, and soft shadows. This plan adds two commerce features:

1. **Subscriptions** — "Subscribe & save" selling plan integration for recurring pet consumables (food, treats, supplements)
2. **Bundles** — "Pet care kit" bundles composed via product metafields, with multi-product add-to-cart

### Shopify prerequisites

- **Subscriptions** require a selling plan app installed on the store (e.g., Shopify Subscriptions, Recharge, Bold Subscriptions). The theme reads `product.selling_plan_groups` which is populated by the app. No app code is written here; the theme only renders selling plan data.
- **Bundles** use product metafields (`custom.bundle_products` — a list of product references) to define which products compose a bundle. The merchant configures these in the Shopify admin under product metafields. Cart additions use the AJAX `/cart/add.js` endpoint with multiple items.

---

## Pages

| Page | Template | Sections |
|---|---|---|
| Product (with subscriptions) | `templates/product.json` | `product-detail` (modified), `subscription-benefits`, `pet-care-bundle`, `related-products` |
| Cart (with subscription/bundle display) | `templates/cart.json` | `cart-content` (modified) |
| Collection (bundle cards) | `templates/collection.json` | `product-grid` (no changes needed; uses `product-card` snippet) |
| Bundles landing page | `templates/page.bundles.json` | `bundle-showcase` |

---

## Sections

| Section | File | Settings | Blocks | Snippets Used |
|---|---|---|---|---|
| **product-detail** (modify) | `sections/product-detail.liquid` | `enable_subscriptions` (checkbox), `subscription_save_percent` (number, display-only fallback), `enable_bundles` (checkbox) | None (new) | `subscription-picker`, `price` |
| **subscription-benefits** (new) | `sections/subscription-benefits.liquid` | `heading` (text), `subheading` (textarea), `background_color` (color) | `benefit_item` blocks: `icon` (select), `heading` (text), `description` (text) | `icon` |
| **pet-care-bundle** (new) | `sections/pet-care-bundle.liquid` | `heading` (text), `subheading` (textarea), `discount_percent` (range 0-30), `discount_label` (text), `bundle_source` (select: metafield/manual), `product_1` through `product_4` (product pickers, for manual mode) | None | `bundle-product-card`, `icon` |
| **bundle-showcase** (new) | `sections/bundle-showcase.liquid` | `heading` (text), `subheading` (textarea) | `bundle_card` blocks: `product` (product picker, the "parent" bundle product), `label` (text), `description` (textarea) | `bundle-showcase-card`, `price`, `icon` |
| **cart-content** (modify) | `sections/cart-content.liquid` | No new settings | None | `icon`, `badge` |

---

## Snippets

| Snippet | File | Parameters | Used By |
|---|---|---|---|
| **subscription-picker** (new) | `snippets/subscription-picker.liquid` | `product` (product object), `current_variant` (variant object), `save_percent_fallback` (number) | `product-detail` |
| **bundle-product-card** (new) | `snippets/bundle-product-card.liquid` | `product` (product object), `checked` (boolean), `index` (number) | `pet-care-bundle` |
| **bundle-showcase-card** (new) | `snippets/bundle-showcase-card.liquid` | `product` (product object), `label` (string), `description` (string) | `bundle-showcase` |
| **icon** (modify) | `snippets/icon.liquid` | Add new icons: `refresh`, `tag`, `package`, `calendar`, `pause` | `subscription-benefits`, `pet-care-bundle`, `cart-content` |
| **badge** (existing) | `snippets/badge.liquid` | Add `subscription` type via CSS | `cart-content` |

---

## Detailed file specifications

### 1. `snippets/subscription-picker.liquid` (NEW)

Renders the "One-time purchase" vs "Subscribe & save" toggle on the product page. Only renders when the product has selling plan groups.

**Parameters:**
- `product` — Shopify product object (required)
- `current_variant` — selected variant (required)
- `save_percent_fallback` — display fallback if selling plan has no price adjustment (default: 10)

**Behavior:**
- Checks `product.selling_plan_groups.size > 0`; if no selling plans, renders nothing
- Renders two radio buttons: "One-time purchase" and "Subscribe & save X%"
- When "Subscribe" is selected, shows a `<select>` with frequency options from `selling_plan_group.selling_plans`
- Each selling plan has `selling_plan.name` (e.g., "Delivery every 2 weeks") and `selling_plan.id`
- Calculates and displays the subscription price using `selling_plan.price_adjustments`
- Inserts/removes a hidden `<input name="selling_plan" value="...">` in the parent form
- Updates the visible price display (`[data-price-container]`) dynamically
- Pet-themed copy: "Never run out of your pet's favorites"

**JavaScript (inline `<script>`):**
- Listens to radio change events to toggle subscription UI visibility
- On frequency select change, updates the hidden `selling_plan` input value
- Calculates adjusted price: if `price_adjustment.type == 'percentage'`, compute `variant_price * (100 - value) / 100`; if `fixed_amount`, subtract; if `price`, use directly
- Updates `[data-price-container]` inner HTML with new price
- Data is serialized into a `<script type="application/json">` block containing the selling plan group data, avoiding repeated Liquid rendering

**CSS (`<style>` tag, since snippets cannot use `{% stylesheet %}`):**
- `.sub-picker` — wrapper
- `.sub-picker__option` — radio button row with label, styled as selectable card (border, rounded, padding)
- `.sub-picker__option--active` — emerald border highlight
- `.sub-picker__frequency` — select dropdown for delivery frequency
- `.sub-picker__savings` — green badge showing savings amount
- `.sub-picker__note` — "Never run out..." pet copy
- Uses PawHaven design tokens: 12px radius, soft shadow, emerald accent

**Schema relevance:** This snippet has no schema (snippets never do). Settings come from the parent section (`product-detail`) via `{% render %}` parameters.

---

### 2. `sections/product-detail.liquid` (MODIFY)

**Changes:**
- Add `enable_subscriptions` checkbox setting (default: `true`)
- Add `subscription_save_percent` number setting (default: `10`, used as display fallback)
- Add `enable_bundles` checkbox setting (default: `true`)
- After the variant selectors and before the quantity selector, conditionally render:
  ```liquid
  {% if section.settings.enable_subscriptions %}
    {% render 'subscription-picker',
      product: product,
      current_variant: current_variant,
      save_percent_fallback: section.settings.subscription_save_percent
    %}
  {% endif %}
  ```
- Modify the AJAX add-to-cart `<script>` to include `selling_plan` from the form data when present
- The existing `FormData` approach already handles this since `FormData` collects all named inputs including the dynamically inserted `selling_plan` hidden input
- Modify the variant change JS to also re-calculate subscription price when variant changes (dispatch a custom event `variant:changed` that the subscription picker listens to)
- Add `window.__pdVariants` data output (it is referenced in the existing JS but never defined; fix this):
  ```liquid
  <script>
    window.__pdVariants = {{ product.variants | json }};
  </script>
  ```

**New schema settings to add:**
```json
{
  "type": "header",
  "content": "t:sections.product_detail.settings.subscription_header.content"
},
{
  "type": "checkbox",
  "id": "enable_subscriptions",
  "label": "t:sections.product_detail.settings.enable_subscriptions.label",
  "default": true,
  "info": "t:sections.product_detail.settings.enable_subscriptions.info"
},
{
  "type": "number",
  "id": "subscription_save_percent",
  "label": "t:sections.product_detail.settings.subscription_save_percent.label",
  "default": 10
},
{
  "type": "header",
  "content": "t:sections.product_detail.settings.bundle_header.content"
},
{
  "type": "checkbox",
  "id": "enable_bundles",
  "label": "t:sections.product_detail.settings.enable_bundles.label",
  "default": true
}
```

---

### 3. `sections/subscription-benefits.liquid` (NEW)

A standalone section that can be added to the product page template (or any page) showcasing the perks of subscribing. Displayed as a horizontal strip with 3-4 benefit cards.

**Settings:**
| Setting | Type | Default |
|---|---|---|
| `heading` | `text` | `"Why subscribe?"` |
| `subheading` | `textarea` | `"Automatic deliveries so your pet never goes without"` |
| `background_color` | `color` | `#F5F0E8` (warm sand) |

**Blocks — `benefit_item` (limit: 6):**
| Setting | Type | Default |
|---|---|---|
| `icon` | `select` | Options: `truck`, `paw`, `tag`, `refresh`, `shield`, `calendar`, `pause`. Default: `truck` |
| `heading` | `text` | (varies per preset) |
| `description` | `text` | (varies per preset) |

**Presets (4 default blocks):**
1. icon: `truck`, heading: "Free shipping", description: "Every subscription delivery ships free"
2. icon: `tag`, heading: "Save on every order", description: "Up to 15% off the regular price"
3. icon: `pause`, heading: "Flexible schedule", description: "Pause, skip, or cancel anytime"
4. icon: `refresh`, heading: "Auto-delivery", description: "Set it and forget it — we will handle the rest"

**Layout:** Centered heading/subheading, then a responsive grid (1 col mobile, 2 col tablet, 4 col desktop) of benefit cards. Each card has icon (emerald), heading (Nunito bold), description (Quicksand).

**CSS (`{% stylesheet %}`):**
- `.sub-benefits` — section wrapper with configurable background via CSS variable
- `.sub-benefits__card` — white card with 12px radius, soft shadow, centered content
- `.sub-benefits__icon` — 48px circle with emerald background, white icon
- Responsive grid

---

### 4. `sections/pet-care-bundle.liquid` (NEW)

Displays a "Complete care kit" bundle on the product page. Reads bundle products from the current product's metafield `custom.bundle_products` (product list reference) or from manual product picker settings.

**Settings:**
| Setting | Type | Default |
|---|---|---|
| `heading` | `text` | `"Complete the care kit"` |
| `subheading` | `textarea` | `"Everything your pet needs in one easy bundle"` |
| `discount_percent` | `range` | min: 0, max: 30, step: 5, default: 10 |
| `discount_label` | `text` | `"Bundle & save"` |
| `bundle_source` | `select` | Options: `metafield` (default), `manual` |
| `product_1` | `product` | blank |
| `product_2` | `product` | blank |
| `product_3` | `product` | blank |
| `product_4` | `product` | blank |

**Logic:**
1. If `bundle_source == 'metafield'`, read `product.metafields.custom.bundle_products.value` (returns array of product objects)
2. If `bundle_source == 'manual'`, collect products from `product_1` through `product_4` settings
3. If no bundle products found, section renders nothing (graceful empty state)
4. Each product is rendered via `bundle-product-card` snippet with a checkbox (default checked)
5. Bundle total is calculated by JS: sum of checked products' first-available-variant prices, then apply discount
6. "Add bundle to cart" button uses AJAX to POST multiple items to `/cart/add.js` with `items[]` array format

**JavaScript (`<script>` tag):**
- Reads product data from a `<script type="application/json">` block (variant IDs, prices)
- Checkbox change events recalculate the bundle total and savings
- "Add bundle to cart" button:
  ```js
  fetch('/cart/add.js', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      items: checkedProducts.map(p => ({
        id: p.variantId,
        quantity: 1,
        properties: { '_bundle': bundleLabel }
      }))
    })
  })
  ```
- Uses line item properties `_bundle` (underscore-prefixed = hidden from customer but available in admin) to group items
- After successful add, updates cart badge and shows toast notification
- Minimum 2 products must be checked to enable the add button

**CSS (`{% stylesheet %}`):**
- `.bundle` — section with sand background
- `.bundle__grid` — responsive grid of bundle product cards (2 col mobile, 4 col desktop)
- `.bundle__summary` — sticky bottom bar with total, savings badge, and add button
- `.bundle__check` — custom checkbox styled with emerald checkmark
- 12px radius on cards, soft shadows

---

### 5. `snippets/bundle-product-card.liquid` (NEW)

Renders a compact product card with a checkbox for bundle selection.

**Parameters:**
- `product` — Shopify product object (required)
- `checked` — boolean, default true
- `index` — number, for unique IDs

**Renders:**
- Checkbox (custom styled) in top-right corner
- Product image (1:1 aspect ratio, 200px)
- Product title (truncated to 2 lines)
- Product price
- Variant info if multiple variants

**CSS (`<style>` tag):**
- `.bundle-card` — compact card, 12px radius
- `.bundle-card__check` — absolute-positioned checkbox
- `.bundle-card__image` — 1:1 aspect ratio
- `.bundle-card__info` — title + price

---

### 6. `sections/bundle-showcase.liquid` (NEW)

A full-width section for a dedicated "Shop bundles" page (`page.bundles.json`). Displays curated bundle products in a grid.

**Settings:**
| Setting | Type | Default |
|---|---|---|
| `heading` | `text` | `"Shop pet care bundles"` |
| `subheading` | `textarea` | `"Curated kits for every pet parent"` |

**Blocks — `bundle_card` (limit: 12):**
| Setting | Type | Default |
|---|---|---|
| `product` | `product` | blank |
| `label` | `text` | blank (e.g., "Puppy starter kit") |
| `description` | `textarea` | blank |

**Layout:** Centered heading, then a responsive grid (1 col mobile, 2 col tablet, 3 col desktop) of bundle showcase cards.

---

### 7. `snippets/bundle-showcase-card.liquid` (NEW)

Renders a large card for the bundle showcase page.

**Parameters:**
- `product` — the "parent" bundle product (required)
- `label` — custom label string (optional, falls back to product title)
- `description` — custom description string (optional)

**Renders:**
- Product featured image
- Label (or product title)
- Description (or product description excerpt)
- "Includes X products" line reading from `product.metafields.custom.bundle_products`
- Thumbnails of included products (up to 4, with "+N more" overflow)
- Price with savings display
- "View bundle" link to product URL

**CSS (`<style>` tag):**
- `.showcase-card` — large card, 12px radius, full image top
- `.showcase-card__includes` — row of small circular thumbnails
- `.showcase-card__price` — prominent price with savings badge

---

### 8. `sections/cart-content.liquid` (MODIFY)

**Changes to the cart line item rendering:**

After the variant title display, add subscription and bundle info:

```liquid
{%- comment -%} Subscription info {%- endcomment -%}
{% if item.selling_plan_allocation %}
  <div class="cart-content__subscription">
    {% assign sub_badge_text = 'cart.subscription_badge' | t %}
    {% render 'badge', text: sub_badge_text, type: 'subscription', small: true %}
    <span class="cart-content__sub-frequency">
      {{ item.selling_plan_allocation.selling_plan.name }}
    </span>
  </div>
{% endif %}

{%- comment -%} Bundle grouping {%- endcomment -%}
{% if item.properties._bundle %}
  <div class="cart-content__bundle-tag">
    {% render 'icon', name: 'package', size: 14 %}
    <span>{{ item.properties._bundle }}</span>
  </div>
{% endif %}
```

**New CSS additions (`{% stylesheet %}` block):**
- `.cart-content__subscription` — flex row with badge and frequency text
- `.cart-content__sub-frequency` — small text, emerald color
- `.cart-content__bundle-tag` — small flex row with package icon and bundle label

---

### 9. `snippets/icon.liquid` (MODIFY)

Add new icon cases to the existing `{% case name %}` block:

| Icon name | Visual | Used for |
|---|---|---|
| `refresh` | Two circular arrows (Lucide refresh-cw) | Auto-delivery benefit |
| `tag` | Price tag (Lucide tag) | Savings benefit |
| `package` | Box/package (Lucide package) | Bundle grouping in cart |
| `calendar` | Calendar (Lucide calendar) | Delivery schedule |
| `pause` | Pause button (Lucide pause-circle) | Pause/skip benefit |

Each icon follows the existing pattern: 24x24 viewBox, stroke-based, `currentColor`, `aria-hidden="true"`, `focusable="false"`.

---

### 10. `snippets/badge.liquid` (MODIFY)

Add a `subscription` badge type CSS variant:

```css
.badge--subscription {
  background-color: var(--color-primary, #059669);
  color: #ffffff;
}
```

No Liquid changes needed; the existing logic already handles arbitrary type values via the `badge--{{ badge_modifier }}` class pattern.

---

### 11. `templates/product.json` (MODIFY)

Add the new sections to the product template:

```json
{
  "sections": {
    "main": {
      "type": "product-detail",
      "settings": {
        "enable_subscriptions": true,
        "subscription_save_percent": 10,
        "enable_bundles": true
      }
    },
    "subscription-benefits": {
      "type": "subscription-benefits",
      "settings": {}
    },
    "bundle": {
      "type": "pet-care-bundle",
      "settings": {}
    },
    "related": {
      "type": "related-products",
      "settings": {}
    }
  },
  "order": ["main", "subscription-benefits", "bundle", "related"]
}
```

---

### 12. `templates/page.bundles.json` (NEW)

```json
{
  "sections": {
    "main": {
      "type": "bundle-showcase",
      "settings": {}
    }
  },
  "order": ["main"]
}
```

---

### 13. `locales/en.default.json` (MODIFY)

Add new translation keys:

```json
{
  "subscriptions": {
    "one_time": "One-time purchase",
    "subscribe_save": "Subscribe & save {{ percent }}%",
    "subscribe_label": "Subscribe & save",
    "frequency_label": "Delivery frequency",
    "per_delivery": "{{ price }} per delivery",
    "savings_note": "You save {{ amount }} per order",
    "never_run_out": "Never run out of your pet's favorites",
    "free_shipping_note": "Free shipping on all subscriptions",
    "next_delivery": "Next delivery: {{ date }}",
    "manage": "Manage subscription"
  },
  "subscription_benefits": {
    "heading": "Why subscribe?",
    "subheading": "Automatic deliveries so your pet never goes without",
    "free_shipping": "Free shipping",
    "free_shipping_desc": "Every subscription delivery ships free",
    "save_on_every_order": "Save on every order",
    "save_desc": "Up to 15% off the regular price",
    "flexible_schedule": "Flexible schedule",
    "flexible_desc": "Pause, skip, or cancel anytime",
    "auto_delivery": "Auto-delivery",
    "auto_delivery_desc": "Set it and forget it -- we will handle the rest"
  },
  "bundles": {
    "heading": "Complete the care kit",
    "subheading": "Everything your pet needs in one easy bundle",
    "bundle_save": "Bundle & save {{ percent }}%",
    "bundle_total": "Bundle total",
    "bundle_savings": "You save {{ amount }}",
    "add_bundle": "Add bundle to cart",
    "select_at_least": "Select at least 2 products",
    "added_to_cart": "Bundle added to cart!",
    "includes_count": "Includes {{ count }} products",
    "view_bundle": "View bundle",
    "shop_bundles_heading": "Shop pet care bundles",
    "shop_bundles_subheading": "Curated kits for every pet parent",
    "no_bundles": "No bundle products configured. Add products in the theme editor."
  },
  "cart": {
    "subscription_badge": "Subscription",
    "delivery_frequency": "Delivery: {{ frequency }}",
    "bundle_label": "Part of bundle"
  }
}
```

Note: The new `cart` keys must be merged into the existing `cart` object, not replace it. The `subscriptions`, `subscription_benefits`, and `bundles` keys are new top-level sections.

---

### 14. `locales/en.default.schema.json` (MODIFY)

Add editor-facing labels for the new sections and settings:

```json
{
  "sections": {
    "product_detail": {
      "settings": {
        "subscription_header": { "content": "Subscriptions" },
        "enable_subscriptions": {
          "label": "Enable subscribe & save",
          "info": "Requires a subscription app (e.g., Shopify Subscriptions) to be installed with selling plans configured on products."
        },
        "subscription_save_percent": { "label": "Savings percentage (display fallback)" },
        "bundle_header": { "content": "Bundles" },
        "enable_bundles": { "label": "Show bundle section on product pages" }
      }
    },
    "subscription_benefits": {
      "name": "Subscription benefits",
      "settings": {
        "heading": { "label": "Heading" },
        "subheading": { "label": "Subheading" },
        "background_color": { "label": "Background color" }
      },
      "blocks": {
        "benefit_item": {
          "name": "Benefit item",
          "settings": {
            "icon": { "label": "Icon" },
            "heading": { "label": "Heading" },
            "description": { "label": "Description" }
          }
        }
      }
    },
    "pet_care_bundle": {
      "name": "Pet care bundle",
      "settings": {
        "heading": { "label": "Heading" },
        "subheading": { "label": "Subheading" },
        "discount_percent": { "label": "Bundle discount (%)" },
        "discount_label": { "label": "Discount label" },
        "bundle_source": { "label": "Bundle product source" },
        "product_1": { "label": "Bundle product 1" },
        "product_2": { "label": "Bundle product 2" },
        "product_3": { "label": "Bundle product 3" },
        "product_4": { "label": "Bundle product 4" }
      }
    },
    "bundle_showcase": {
      "name": "Bundle showcase",
      "settings": {
        "heading": { "label": "Heading" },
        "subheading": { "label": "Subheading" }
      },
      "blocks": {
        "bundle_card": {
          "name": "Bundle card",
          "settings": {
            "product": { "label": "Bundle product" },
            "label": { "label": "Bundle label" },
            "description": { "label": "Description" }
          }
        }
      }
    }
  }
}
```

Note: These keys must be merged into the existing schema JSON, not replace it. Add new keys under the existing `sections` object.

---

### 15. `assets/critical.css` (MODIFY)

No additions required. All new component styles live in their respective section `{% stylesheet %}` blocks or snippet `<style>` tags. The existing `.btn`, `.card`, `.badge`, and `.field` classes are already available globally.

---

## JavaScript approach

All JavaScript is vanilla JS in `<script>` tags (per CLAUDE.md conventions). No frameworks or libraries.

### Subscription picker JS (in `subscription-picker.liquid`)
- Reads selling plan data from inline `<script type="application/json">` block
- Listens to `change` events on radio buttons and frequency select
- Dynamically inserts/removes `<input type="hidden" name="selling_plan">` in the parent form
- Dispatches `subscription:changed` custom event with selected plan details
- Listens for `variant:changed` custom event to recalculate subscription price

### Bundle section JS (in `pet-care-bundle.liquid`)
- Reads bundle product data from inline `<script type="application/json">` block
- Checkbox change handlers recalculate total and savings
- "Add bundle to cart" uses `fetch('/cart/add.js')` with `items[]` array
- After success: dispatches `cart:updated` event, shows toast, updates cart badge
- Validates minimum 2 products selected before enabling add button

### Product detail JS modifications (in `product-detail.liquid`)
- Adds `window.__pdVariants = {{ product.variants | json }};` output
- On variant change, dispatches `variant:changed` custom event with the new variant object
- The existing form submit handler already works with `FormData` which captures the `selling_plan` input

### Cart JS (in `cart-content.liquid`)
- No JS changes needed; the subscription and bundle display is server-rendered Liquid

---

## Metafield setup (merchant configuration)

The merchant needs to create one product metafield in Shopify admin:

| Namespace | Key | Type | Description |
|---|---|---|---|
| `custom` | `bundle_products` | `list.product_reference` | List of product handles that compose this bundle. Maximum 4 products. |

This is configured in Settings > Custom data > Products > Add definition.

---

## Build order

| # | File | Reason |
|---|---|---|
| 1 | `snippets/icon.liquid` | Add 5 new icons (refresh, tag, package, calendar, pause). No dependencies. |
| 2 | `snippets/badge.liquid` | Add `.badge--subscription` CSS. No dependencies. |
| 3 | `locales/en.default.json` | Add all new translation keys. Must be done before any Liquid that references them. |
| 4 | `locales/en.default.schema.json` | Add all new editor labels. Must be done before schema references. |
| 5 | `snippets/subscription-picker.liquid` | New file. Depends on #1 (icons) and #3 (translations). |
| 6 | `snippets/bundle-product-card.liquid` | New file. Depends on #3 (translations). |
| 7 | `snippets/bundle-showcase-card.liquid` | New file. Depends on #2 (badge), #3 (translations). |
| 8 | `sections/product-detail.liquid` | Modify to add subscription picker render call, new schema settings, and `__pdVariants` output. Depends on #3, #4, #5. |
| 9 | `sections/subscription-benefits.liquid` | New section. Depends on #1, #3, #4. |
| 10 | `sections/pet-care-bundle.liquid` | New section. Depends on #1, #3, #4, #6. |
| 11 | `sections/bundle-showcase.liquid` | New section. Depends on #3, #4, #7. |
| 12 | `sections/cart-content.liquid` | Modify to show subscription/bundle info on line items. Depends on #1, #2, #3. |
| 13 | `templates/product.json` | Add new sections to product template. Depends on #8, #9, #10. |
| 14 | `templates/page.bundles.json` | New template for bundles landing page. Depends on #11. |
| 15 | Validation and testing | Run `validate_theme`, test with dev store. |

---

## Translation keys (full hierarchy)

```
subscriptions/
  one_time
  subscribe_save          (interpolation: percent)
  subscribe_label
  frequency_label
  per_delivery            (interpolation: price)
  savings_note            (interpolation: amount)
  never_run_out
  free_shipping_note
  next_delivery           (interpolation: date)
  manage

subscription_benefits/
  heading
  subheading
  free_shipping
  free_shipping_desc
  save_on_every_order
  save_desc
  flexible_schedule
  flexible_desc
  auto_delivery
  auto_delivery_desc

bundles/
  heading
  subheading
  bundle_save             (interpolation: percent)
  bundle_total
  bundle_savings          (interpolation: amount)
  add_bundle
  select_at_least
  added_to_cart
  includes_count          (interpolation: count)
  view_bundle
  shop_bundles_heading
  shop_bundles_subheading
  no_bundles

cart/                     (merge into existing)
  subscription_badge
  delivery_frequency      (interpolation: frequency)
  bundle_label
```

---

## Files summary

### New files (7)
| File | Type |
|---|---|
| `snippets/subscription-picker.liquid` | Snippet |
| `snippets/bundle-product-card.liquid` | Snippet |
| `snippets/bundle-showcase-card.liquid` | Snippet |
| `sections/subscription-benefits.liquid` | Section |
| `sections/pet-care-bundle.liquid` | Section |
| `sections/bundle-showcase.liquid` | Section |
| `templates/page.bundles.json` | Template |

### Modified files (7)
| File | Changes |
|---|---|
| `snippets/icon.liquid` | Add 5 new icon cases |
| `snippets/badge.liquid` | Add `.badge--subscription` CSS |
| `sections/product-detail.liquid` | Add subscription picker render, new settings, `__pdVariants` output, variant change event |
| `sections/cart-content.liquid` | Add subscription badge, frequency display, bundle grouping tag |
| `templates/product.json` | Add subscription-benefits and pet-care-bundle sections |
| `locales/en.default.json` | Add subscriptions, subscription_benefits, bundles keys; merge cart keys |
| `locales/en.default.schema.json` | Add section/block editor labels |

### Unchanged files
All other existing files remain untouched. No changes to `layout/theme.liquid`, `config/settings_schema.json`, or `assets/critical.css`.

---

## Design integration notes

All new components follow PawHaven design tokens:

- **Colors:** Emerald (#059669) for active states, savings badges, icons. Warm sand (#F5F0E8) for section backgrounds. Amber (#D97706) for sale/discount badges.
- **Typography:** Nunito (headings, buttons, labels), Quicksand (body text, descriptions).
- **Border radius:** 12px on all cards and containers. Pill radius (100px) on badges and radio-style buttons.
- **Shadows:** `var(--shadow-soft)` on cards, `var(--shadow-hover)` on hover states.
- **Pet-themed copy:** "Never run out of your pet's favorites", "Complete the care kit", paw-print iconography in benefits section.
- **Responsive:** Mobile-first. Single column on mobile, 2-column on tablet, 4-column on desktop for grids.
- **Accessibility:** All interactive elements have `aria-label` or visible text labels. Custom checkboxes include hidden native `<input type="checkbox">` for screen reader support. Focus-visible outlines in emerald. Radio groups use `role="radiogroup"`.

---

## Testing checklist

- [ ] Product page without selling plans: subscription picker hidden, no errors
- [ ] Product page with selling plans: toggle between one-time and subscribe, price updates correctly
- [ ] Subscribe & save adds to cart with selling_plan ID in request body
- [ ] Cart shows subscription badge and frequency for subscription line items
- [ ] Product page with bundle metafield: bundle section shows bundled products
- [ ] Product page without bundle metafield: bundle section hidden
- [ ] Bundle checkboxes toggle products in/out, total recalculates
- [ ] "Add bundle to cart" adds all checked products with `_bundle` property
- [ ] Cart groups bundle items with bundle tag display
- [ ] Bundles landing page renders bundle showcase cards
- [ ] Mobile responsive layout for all new sections
- [ ] Keyboard navigation through subscription radio, frequency select, bundle checkboxes
- [ ] Screen reader announces subscription option changes
- [ ] `validate_theme` passes with no errors
- [ ] Theme editor: all new settings are labeled and functional
