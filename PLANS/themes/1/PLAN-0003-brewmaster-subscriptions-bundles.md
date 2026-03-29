# PLAN-0003: BrewMaster Subscriptions and Bundles

**Theme:** BrewMaster (`themes/starter-minimal`)
**Status:** Planned
**Estimated effort:** 3-4 days
**ADR reference:** ADR-0002, Project 12

---

## Context

The BrewMaster theme currently supports standard one-time purchases with wholesale/B2B pricing. This plan adds two commerce features that are explicitly required by the portfolio showcase strategy (ADR-0002, gaps #13 and #14):

1. **Subscriptions** -- Selling plan integration for consumable products (coffee beans, brewing supplies) allowing customers to subscribe and save with recurring delivery.
2. **Bundles** -- "Frequently bought together" and curated bundle functionality using product metafields and AJAX cart additions.

### Prerequisites

- A subscription app must be installed on the store (e.g., Shopify Subscriptions app) to create selling plan groups via the Admin API. The theme code reads `product.selling_plan_groups` which is populated by the app.
- Bundle product relationships are stored in product metafields (`custom.bundle_products` -- a list of product references).
- Bundle discounts are display-only in the theme (like wholesale pricing). Actual enforcement requires Shopify automatic discounts or Scripts.

---

## Pages

| Page | Template | Sections |
|---|---|---|
| Product (existing) | `product.json` | `product-detail` (modified), `product-bundle`, `wholesale-pricing-table`, `product-tabs`, `related-products` |
| Cart (existing) | `cart.json` | `cart-content` (modified) |
| Bundle landing | `page.bundles.json` (new) | `bundle-landing-hero`, `bundle-showcase` |

---

## Sections

| Section | New/Modified | Settings | Blocks | Snippets Used |
|---|---|---|---|---|
| `product-detail` | Modified | + `show_subscription_options` (checkbox) | -- | `subscription-option`, `price`, `badge`, `icon` |
| `product-bundle` | New | `heading`, `discount_percent`, `max_products` | -- | `product-card`, `icon`, `price` |
| `cart-content` | Modified | + `show_subscription_details`, `show_bundle_grouping` (checkboxes) | -- | `subscription-cart-badge`, `bundle-cart-group`, `icon` |
| `bundle-landing-hero` | New | `heading`, `subtitle`, `image`, `button_label`, `button_url` | -- | `button`, `icon` |
| `bundle-showcase` | New | `heading` | `bundle_card` (block) | `product-card`, `price`, `icon` |

---

## Snippets

| Snippet | New/Modified | Parameters | Used By |
|---|---|---|---|
| `subscription-option` | New | `product` (product object) | `product-detail` |
| `subscription-info` | New | `selling_plan` (selling plan object), `show_savings` (boolean), `savings_percent` (number) | `subscription-option`, `subscription-cart-badge` |
| `subscription-cart-badge` | New | `line_item` (line item object) | `cart-content` |
| `bundle-cart-group` | New | `cart` (cart object) | `cart-content` |
| `icon` | Modified | + new icon names: `repeat`, `package`, `tag` | All subscription/bundle components |

---

## Detailed Component Specifications

### 1. Subscription Option on Product Page

**File:** `snippets/subscription-option.liquid`

This snippet renders inside the product form in `product-detail.liquid`, between the price display and the quantity selector. It reads `product.selling_plan_groups` and renders a purchase option selector.

**Liquid data flow:**
```
product.selling_plan_groups -> for each group:
  group.name (e.g., "Subscribe and save")
  group.selling_plans -> for each plan:
    plan.id
    plan.name (e.g., "Delivery every 2 weeks")
    plan.price_adjustments -> adjustment.value (percentage or fixed)
    plan.options -> option.name, option.value (e.g., "Delivery every", "2 Weeks")
```

**UI structure:**
- Radio button group: "One-time purchase" (default selected) vs "Subscribe & save X%"
- When subscription is selected, a `<select>` dropdown appears showing delivery frequency options from `group.selling_plans`
- Price display updates dynamically (JS) to show per-delivery price
- A hidden input `<input type="hidden" name="selling_plan" value="">` is included in the product form. Value is empty for one-time, set to `plan.id` for subscription.
- Below the selector, renders `subscription-info` snippet showing savings and delivery details.

**Schema settings (on product-detail section):**
```json
{
  "type": "checkbox",
  "id": "show_subscription_options",
  "label": "t:sections.product_detail.settings.show_subscription_options.label",
  "default": true
}
```

**JavaScript behavior:**
- On radio change to "subscribe": show frequency dropdown, update hidden `selling_plan` input, recalculate displayed price
- On radio change to "one-time": hide frequency dropdown, clear `selling_plan` input, restore original price
- On frequency dropdown change: update `selling_plan` input value, recalculate price with that plan's adjustment
- Price calculation: read `data-original-price` attribute (cents), apply percentage adjustment from `data-plan-adjustments` JSON

**Interaction with wholesale pricing:**
- When a wholesale customer views the product, the subscription discount applies to the *retail* price (not the wholesale price). The UI should make clear which base price the subscription discount applies to.
- If both wholesale and subscription are available, show a note: "Subscription pricing is based on the retail price."

### 2. Subscription Info Snippet

**File:** `snippets/subscription-info.liquid`

**Parameters:**
| Param | Type | Required | Description |
|---|---|---|---|
| `selling_plan` | selling_plan | No | Current selling plan object. When blank, snippet renders nothing. |
| `show_savings` | boolean | No | Show savings badge. Default: true. |
| `savings_percent` | number | No | Override savings percentage for display. |

**Renders:**
- Delivery frequency: truck icon + "Delivered every 2 weeks"
- Savings badge: tag icon + "Save 15% per delivery" (rendered via `badge` snippet with type `subscription`)
- Flexibility note: check icon + "Cancel or pause anytime"
- Commitment info: if the selling plan has a fixed commitment, show "3-delivery minimum" or similar

### 3. Subscription Cart Badge

**File:** `snippets/subscription-cart-badge.liquid`

**Parameters:**
| Param | Type | Required | Description |
|---|---|---|---|
| `line_item` | line_item | Yes | Cart line item to check for subscription data. |

**Logic:**
- Check `line_item.selling_plan_allocation` -- if present, the item is a subscription
- Display a small badge with repeat icon + delivery frequency text
- Show per-delivery price vs one-time price comparison

**Renders:**
```
[repeat icon] Subscription: Every 2 weeks
Per delivery: $25.49 (was $29.99)
```

### 4. Product Bundle Section

**File:** `sections/product-bundle.liquid`

This is a new section that reads bundle data from the current product's metafield `custom.bundle_products` (a list of product references).

**Schema settings:**
```json
{
  "type": "text",
  "id": "heading",
  "label": "t:sections.product_bundle.settings.heading.label",
  "default": "Frequently bought together"
},
{
  "type": "range",
  "id": "discount_percent",
  "label": "t:sections.product_bundle.settings.discount_percent.label",
  "min": 0,
  "max": 30,
  "step": 5,
  "unit": "%",
  "default": 10
},
{
  "type": "range",
  "id": "max_products",
  "label": "t:sections.product_bundle.settings.max_products.label",
  "min": 2,
  "max": 4,
  "step": 1,
  "default": 3
}
```

**Data flow:**
```
product.metafields.custom.bundle_products -> list of product references
  for each bundle_product:
    bundle_product.title
    bundle_product.price
    bundle_product.first_available_variant.id
    bundle_product.featured_image
```

**UI structure:**
- Section heading: "Frequently bought together" (customizable)
- Current product card (always included, checkbox checked and disabled)
- 2-3 additional product cards from the metafield, each with a checkbox (default checked)
- Plus signs (+) between product cards
- Price summary panel:
  - Individual prices for checked items
  - Bundle total with discount applied
  - Savings amount: "Save $X.XX (10% off)"
- "Add bundle to cart" button -- adds all checked items via AJAX `/cart/add.js` with `items[]` array

**JavaScript behavior:**
- Checkbox toggle: recalculate total and savings display
- "Add bundle to cart" button: POST to `/cart/add.js` with `{ items: [...] }` format
- After successful add: show toast notification (reuse existing toast from product-detail), update cart badge
- Bundle discount is display-only -- the actual discount must be enforced by an automatic discount in Shopify admin (e.g., "Buy X products from collection Y, get 10% off")

**Conditional rendering:**
- Section only renders if `product.metafields.custom.bundle_products` is not blank
- If metafield has products but none are available, section hides

### 5. Bundle Landing Page

**Template:** `templates/page.bundles.json`

Composes two new sections for a dedicated bundles page.

#### 5a. Bundle Landing Hero Section

**File:** `sections/bundle-landing-hero.liquid`

**Schema settings:**
```json
{
  "type": "image_picker",
  "id": "image",
  "label": "t:sections.bundle_landing_hero.settings.image.label"
},
{
  "type": "text",
  "id": "heading",
  "label": "t:sections.bundle_landing_hero.settings.heading.label",
  "default": "Curated brewing bundles"
},
{
  "type": "textarea",
  "id": "subtitle",
  "label": "t:sections.bundle_landing_hero.settings.subtitle.label",
  "default": "Everything you need to start brewing, bundled together at a discount."
}
```

**Renders:**
- Full-width hero with background image, heading, and subtitle
- Follows same pattern as existing `hero-banner.liquid` but simpler (no CTA buttons)

#### 5b. Bundle Showcase Section

**File:** `sections/bundle-showcase.liquid`

Uses blocks to let merchants define individual bundles in the theme editor.

**Schema:**
```json
{
  "name": "t:sections.bundle_showcase.name",
  "settings": [
    {
      "type": "text",
      "id": "heading",
      "label": "t:sections.bundle_showcase.settings.heading.label",
      "default": "Shop our bundles"
    }
  ],
  "blocks": [
    {
      "type": "bundle_card",
      "name": "t:blocks.bundle_card.name",
      "settings": [
        {
          "type": "text",
          "id": "title",
          "label": "t:blocks.bundle_card.settings.title.label",
          "default": "Starter kit"
        },
        {
          "type": "textarea",
          "id": "description",
          "label": "t:blocks.bundle_card.settings.description.label"
        },
        {
          "type": "image_picker",
          "id": "image",
          "label": "t:blocks.bundle_card.settings.image.label"
        },
        {
          "type": "product_list",
          "id": "products",
          "label": "t:blocks.bundle_card.settings.products.label",
          "limit": 4
        },
        {
          "type": "range",
          "id": "discount_percent",
          "label": "t:blocks.bundle_card.settings.discount_percent.label",
          "min": 0,
          "max": 30,
          "step": 5,
          "unit": "%",
          "default": 10
        },
        {
          "type": "url",
          "id": "url",
          "label": "t:blocks.bundle_card.settings.url.label"
        }
      ]
    }
  ],
  "presets": [
    {
      "name": "Bundle showcase"
    }
  ]
}
```

**Renders per block:**
- Bundle card with image, title, description
- List of included product names with small thumbnails
- Total price (sum of products), savings amount, discounted total
- "Shop this bundle" CTA button linking to `block.settings.url` (typically the first product's page, or a custom page)

### 6. Cart Bundle Summary

**File:** `snippets/bundle-cart-group.liquid`

**Parameters:**
| Param | Type | Required | Description |
|---|---|---|---|
| `cart` | cart | Yes | The cart object. |

**Logic:**
- Identify bundle items by checking `line_item.properties._bundle_id` (a unique string set by the JS when adding bundle items)
- Group items with matching `_bundle_id` visually
- Show a "Bundle" label and discount indicator on the group
- Bundle items that were added individually (not through the bundle button) will not have `_bundle_id` and render normally

**Renders:**
```
[package icon] Bundle
  - Product A    $29.99
  - Product B    $19.99
  Bundle savings: -$5.00 (10% off)
```

---

## File Changes Summary

### New Files (12 files)

| # | File | Type | Description |
|---|---|---|---|
| 1 | `snippets/subscription-option.liquid` | Snippet | Purchase option selector (one-time vs subscribe) |
| 2 | `snippets/subscription-info.liquid` | Snippet | Delivery frequency, savings badge, cancel info |
| 3 | `snippets/subscription-cart-badge.liquid` | Snippet | Cart line item subscription indicator |
| 4 | `snippets/bundle-cart-group.liquid` | Snippet | Cart bundle item grouping |
| 5 | `sections/product-bundle.liquid` | Section | "Frequently bought together" on product pages |
| 6 | `sections/bundle-landing-hero.liquid` | Section | Hero banner for bundle landing page |
| 7 | `sections/bundle-showcase.liquid` | Section | Grid of curated bundle cards |
| 8 | `templates/page.bundles.json` | Template | Bundle landing page template |

### Modified Files (7 files)

| # | File | Change |
|---|---|---|
| 1 | `sections/product-detail.liquid` | Add subscription option rendering, `selling_plan` hidden input, JS for dynamic pricing |
| 2 | `sections/cart-content.liquid` | Add subscription badge and bundle grouping rendering, new section settings |
| 3 | `snippets/icon.liquid` | Add `repeat`, `package`, `tag` icons |
| 4 | `snippets/badge.liquid` | Add `subscription` and `bundle` badge variants |
| 5 | `config/settings_schema.json` | No global settings changes needed (all config is per-section) |
| 6 | `locales/en.default.json` | Add `subscriptions.*` and `bundles.*` translation keys |
| 7 | `locales/en.default.schema.json` | Add schema labels for new sections, blocks, and settings |
| 8 | `templates/product.json` | Add `product-bundle` section to order |

---

## Detailed Modifications

### product-detail.liquid Changes

**Add after the price display block (line ~41) and before the stock display:**

```liquid
{% if section.settings.show_subscription_options %}
  {% if product.selling_plan_groups.size > 0 %}
    {% render 'subscription-option', product: product %}
  {% endif %}
{% endif %}
```

**Add inside the product form (after the hidden `id` input, before quantity):**

```liquid
<input type="hidden" name="selling_plan" value="" data-selling-plan-input>
```

**Modify the form submit JS** to include the `selling_plan` value from the hidden input in the POST body. The existing `FormData` approach already handles this since it reads all form inputs, but verify the hidden input is captured.

**Add new schema setting:**

```json
{
  "type": "checkbox",
  "id": "show_subscription_options",
  "label": "t:sections.product_detail.settings.show_subscription_options.label",
  "default": true
}
```

### cart-content.liquid Changes

**Add after the variant title display (line ~25) inside each line item:**

```liquid
{% if section.settings.show_subscription_details %}
  {% render 'subscription-cart-badge', line_item: item %}
{% endif %}
```

**Add before the totals section, after wholesale savings:**

```liquid
{% if section.settings.show_bundle_grouping %}
  {% render 'bundle-cart-group', cart: cart %}
{% endif %}
```

**Add new schema settings:**

```json
{
  "type": "checkbox",
  "id": "show_subscription_details",
  "label": "t:sections.cart_content.settings.show_subscription_details.label",
  "default": true
},
{
  "type": "checkbox",
  "id": "show_bundle_grouping",
  "label": "t:sections.cart_content.settings.show_bundle_grouping.label",
  "default": true
}
```

### product.json Changes

Add the `product-bundle` section between `wholesale_pricing` and `tabs`:

```json
{
  "sections": {
    "main": { "type": "product-detail", "settings": {} },
    "wholesale_pricing": { ... },
    "bundle": {
      "type": "product-bundle",
      "settings": {
        "heading": "Frequently bought together",
        "discount_percent": 10,
        "max_products": 3
      }
    },
    "tabs": { ... },
    "related": { ... }
  },
  "order": ["main", "wholesale_pricing", "bundle", "tabs", "related"]
}
```

### icon.liquid Changes

Add three new icon cases:

- `repeat` -- circular arrows icon (for subscription/recurring)
- `package` -- box/package icon (for bundles)
- `tag` -- price tag icon (for savings/discounts)

### badge.liquid Changes

Add two new badge variants:

```liquid
{% elsif type == 'subscription' %}
  <span class="badge badge--subscription">{{ label }}</span>
{% elsif type == 'bundle' %}
  <span class="badge badge--bundle">{{ label }}</span>
```

With CSS in the snippet:

```css
.badge--subscription {
  background: var(--color-success);
  color: #ffffff;
}

.badge--bundle {
  background: var(--color-primary);
  color: var(--color-primary-contrast);
}
```

---

## Translation Keys

### locales/en.default.json additions

```json
{
  "subscriptions": {
    "one_time_purchase": "One-time purchase",
    "subscribe_and_save": "Subscribe & save {{ percent }}%",
    "select_frequency": "Delivery frequency",
    "per_delivery": "{{ price }} per delivery",
    "you_save": "You save {{ amount }} per delivery",
    "cancel_anytime": "Cancel or pause anytime",
    "commitment_notice": "{{ count }}-delivery minimum commitment",
    "subscription_label": "Subscription",
    "delivery_every": "Delivered every {{ interval }}",
    "savings_badge": "Save {{ percent }}%",
    "based_on_retail": "Subscription pricing is based on the retail price"
  },
  "bundles": {
    "frequently_bought_together": "Frequently bought together",
    "complete_the_set": "Complete the set",
    "bundle_total": "Bundle total",
    "bundle_savings": "You save {{ amount }}",
    "bundle_discount": "{{ percent }}% off when bought together",
    "add_bundle_to_cart": "Add bundle to cart",
    "adding_bundle": "Adding...",
    "bundle_added": "Bundle added to cart",
    "bundle_error": "Could not add bundle to cart",
    "this_item": "This item",
    "included_in_bundle": "Part of a bundle",
    "bundle_label": "Bundle",
    "shop_this_bundle": "Shop this bundle",
    "curated_bundles_heading": "Curated brewing bundles",
    "curated_bundles_subtitle": "Everything you need to start brewing, bundled together at a discount.",
    "items_included": "{{ count }} items included"
  }
}
```

### locales/en.default.schema.json additions

```json
{
  "sections": {
    "product_detail": {
      "settings": {
        "show_subscription_options": { "label": "Show subscription options" }
      }
    },
    "product_bundle": {
      "name": "Product bundle",
      "settings": {
        "heading": { "label": "Heading" },
        "discount_percent": { "label": "Bundle discount percentage" },
        "max_products": { "label": "Maximum products in bundle" }
      }
    },
    "cart_content": {
      "settings": {
        "show_subscription_details": { "label": "Show subscription details on line items" },
        "show_bundle_grouping": { "label": "Show bundle item grouping" }
      }
    },
    "bundle_landing_hero": {
      "name": "Bundle landing hero",
      "settings": {
        "image": { "label": "Background image" },
        "heading": { "label": "Heading" },
        "subtitle": { "label": "Subtitle" }
      }
    },
    "bundle_showcase": {
      "name": "Bundle showcase",
      "settings": {
        "heading": { "label": "Heading" }
      }
    }
  },
  "blocks": {
    "bundle_card": {
      "name": "Bundle card",
      "settings": {
        "title": { "label": "Bundle title" },
        "description": { "label": "Description" },
        "image": { "label": "Bundle image" },
        "products": { "label": "Products in bundle" },
        "discount_percent": { "label": "Discount percentage" },
        "url": { "label": "Link URL" }
      }
    }
  }
}
```

---

## JavaScript Approach

### Subscription Price Switcher (in product-detail.liquid)

The subscription option snippet outputs a JSON data blob in a `<script type="application/json">` tag containing all selling plan data for the product:

```json
{
  "groups": [
    {
      "id": "gid://...",
      "name": "Subscribe and save",
      "plans": [
        {
          "id": 123456,
          "name": "Delivery every 2 weeks",
          "price_adjustment_value": 15,
          "price_adjustment_type": "percentage"
        }
      ]
    }
  ],
  "original_price_cents": 2999
}
```

**JS logic (vanilla, in `<script>` tag within `subscription-option.liquid`):**
1. Listen for `change` events on the purchase option radio buttons
2. When "subscribe" is selected, show the frequency `<select>`, update hidden `selling_plan` input
3. Calculate adjusted price: `original_price * (1 - adjustment/100)` for percentage adjustments
4. Update the `.product-detail__price` element text content
5. When "one-time" is selected, restore original price, clear `selling_plan` input

### Bundle Add-to-Cart (in product-bundle.liquid)

**JS logic (vanilla, in `<script>` tag within `product-bundle.liquid`):**
1. Listen for checkbox `change` events to recalculate totals
2. On "Add bundle to cart" click:
   - Collect all checked product variant IDs
   - Generate a unique `_bundle_id` (timestamp-based)
   - POST to `/cart/add.js` with:
     ```json
     {
       "items": [
         {
           "id": 12345,
           "quantity": 1,
           "properties": { "_bundle_id": "bundle_1711612800" }
         },
         {
           "id": 67890,
           "quantity": 1,
           "properties": { "_bundle_id": "bundle_1711612800" }
         }
       ]
     }
     ```
   - On success: show toast, update cart badge via `cart:updated` event
   - On failure: show error toast

Note: the `_bundle_id` line property is prefixed with underscore so Shopify hides it from the customer in checkout. It is used purely for visual grouping in the cart template.

---

## Build Order

| # | File | Depends On | Reason |
|---|---|---|---|
| 1 | `locales/en.default.json` | -- | Translation keys needed by all components |
| 2 | `locales/en.default.schema.json` | -- | Schema labels needed by all sections |
| 3 | `snippets/icon.liquid` | -- | New icons needed by subscription and bundle components |
| 4 | `snippets/badge.liquid` | -- | New badge variants needed by subscription and bundle components |
| 5 | `snippets/subscription-info.liquid` | #1, #3, #4 | Reusable info display, used by option snippet and cart badge |
| 6 | `snippets/subscription-option.liquid` | #1, #3, #5 | Purchase option selector, used by product-detail |
| 7 | `snippets/subscription-cart-badge.liquid` | #1, #3, #5 | Cart line item badge, used by cart-content |
| 8 | `sections/product-detail.liquid` | #6 | Integrate subscription option into existing product form |
| 9 | `sections/product-bundle.liquid` | #1, #3 | New section for product page bundles |
| 10 | `templates/product.json` | #9 | Add product-bundle section to product template |
| 11 | `snippets/bundle-cart-group.liquid` | #1, #3, #4 | Cart bundle grouping display |
| 12 | `sections/cart-content.liquid` | #7, #11 | Integrate subscription badges and bundle grouping |
| 13 | `sections/bundle-landing-hero.liquid` | #1 | Hero for bundle landing page |
| 14 | `sections/bundle-showcase.liquid` | #1, #3, #4 | Bundle cards grid for landing page |
| 15 | `templates/page.bundles.json` | #13, #14 | Bundle landing page template |
| 16 | `assets/critical.css` | -- | Add shared subscription/bundle CSS if needed |

---

## Metafield Configuration

The following metafields must be created in the Shopify admin (Settings > Custom data > Products):

| Namespace | Key | Type | Description |
|---|---|---|---|
| `custom` | `bundle_products` | `list.product_reference` | Products to show in the "Frequently bought together" section. Max 4 entries. |

This metafield is set per-product by the merchant in the product admin page.

---

## Mobile Responsiveness

### Subscription Option (product page)
- Radio buttons stack vertically on all screen sizes (already natural flow)
- Frequency dropdown is full-width on mobile
- Savings badge wraps below the price on narrow screens

### Product Bundle Section
- Products display in a 2-column grid on mobile, expanding to a horizontal row on desktop
- Plus signs (+) between products hide on mobile (replaced with vertical stacking)
- Price summary panel goes full-width below the product grid on mobile, sits alongside on desktop
- "Add bundle to cart" button is full-width on mobile

### Bundle Landing Page
- Bundle cards stack in a single column on mobile, 2 columns on tablet, 3 on desktop
- Product thumbnails within bundle cards use a compact horizontal scroll on mobile

### Cart Displays
- Subscription badge renders inline below the product title on all sizes
- Bundle grouping uses a subtle left border indent on mobile

---

## Accessibility

- Purchase option radio buttons use proper `<fieldset>` and `<legend>` elements
- Frequency `<select>` has a visible `<label>`
- Bundle product checkboxes have associated `<label>` elements with product titles
- Bundle price summary uses `aria-live="polite"` so screen readers announce changes
- All new icons have `aria-hidden="true"` (decorative, text labels always present)
- Bundle "Add to cart" button has loading state with `aria-busy="true"`
- Cart subscription badge text is readable without the icon

---

## Testing Checklist

### Subscriptions
- [ ] Product with selling plans shows one-time/subscribe toggle
- [ ] Product without selling plans shows no subscription UI
- [ ] Switching between one-time and subscribe updates the displayed price
- [ ] Changing frequency updates the `selling_plan` hidden input value
- [ ] Add to cart with subscription submits correct `selling_plan` ID
- [ ] Cart shows subscription badge with frequency on subscribed items
- [ ] Cart shows no subscription badge on one-time items
- [ ] Wholesale customer sees note about subscription pricing base
- [ ] Mobile layout is usable at 320px width

### Bundles
- [ ] Product with `custom.bundle_products` metafield shows bundle section
- [ ] Product without metafield hides bundle section entirely
- [ ] Unchecking a product updates the bundle total and savings
- [ ] "Add bundle to cart" adds all checked items with `_bundle_id` property
- [ ] Toast notification appears after successful bundle add
- [ ] Cart badge count updates after bundle add
- [ ] Cart groups bundle items visually when `_bundle_id` matches
- [ ] Bundle landing page renders with merchant-configured bundles
- [ ] Bundle card shows correct total price and savings
- [ ] Mobile layout is usable at 320px width

### Integration
- [ ] Subscription + bundle on same product page both render correctly
- [ ] Wholesale customer can subscribe (UI shows retail price note)
- [ ] Theme editor can toggle all new features on/off via checkboxes
- [ ] All text uses translation keys (no hardcoded English strings)
- [ ] All new sections have presets and appear in theme editor "Add section"
