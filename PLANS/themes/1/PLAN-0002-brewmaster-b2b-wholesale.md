# PLAN-0002: B2B Wholesale Pricing System for BrewMaster

**Theme:** BrewMaster (`themes/starter-minimal`)
**ADR reference:** ADR-0002, Project 8
**Effort estimate:** 2-3 days
**Purpose:** Demonstrate B2B wholesale pricing logic using customer tags, metafields, and tiered quantity breaks. This is a display/UX layer implementation. Actual price enforcement at checkout requires Shopify Scripts (Plus), automatic discounts, or discount codes, which is documented but not implemented in Liquid.

---

## Architecture Overview

### How Wholesale Pricing Works in This Implementation

1. **Customer tags** determine pricing tier: `wholesale`, `vip`, `distributor`
2. **Product metafields** store per-product wholesale prices and quantity break rules
3. **Liquid conditionals** show/hide pricing based on `customer.tags`
4. **JavaScript** handles dynamic UI updates (quick order form, cart validation, quantity enforcement)
5. **Shopify automatic discounts or Scripts** enforce actual pricing at checkout (documented, not built in Liquid)

### Customer Tiers

| Tag | Tier Name | Typical Discount | Min Order Qty |
|---|---|---|---|
| `wholesale` | Wholesale | 15% off retail | 10 units |
| `vip` | VIP | 20% off retail | 5 units |
| `distributor` | Distributor | 25% off retail | 25 units |

These are default values. Merchants configure tier names, discount percentages, and minimum quantities in global theme settings.

---

## Pages

| Page | Template | Sections |
|---|---|---|
| Product (existing) | `templates/product.json` | product-detail (modified), wholesale-pricing-table (new), product-tabs, related-products |
| Collection (existing) | `templates/collection.json` | product-grid (modified via product-card snippet) |
| Cart (existing) | `templates/cart.json` | cart-content (modified) |
| Wholesale application | `templates/page.wholesale-application.json` (new) | wholesale-application-form (new) |
| Quick order | `templates/page.quick-order.json` (new) | quick-order-form (new) |

---

## Sections

| Section | New/Modified | Settings | Blocks | Snippets Used |
|---|---|---|---|---|
| `product-detail` | Modified | + `show_wholesale_pricing` (checkbox) | None | `price`, `wholesale-price`, `badge`, `quantity-selector` |
| `wholesale-pricing-table` | New | `heading` (text) | `quantity_break` (repeatable) | `wholesale-badge` |
| `cart-content` | Modified | + `show_wholesale_savings` (checkbox) | None | `wholesale-cart-summary` |
| `product-grid` | Modified (indirectly via snippet) | None | None | `product-card` (modified) |
| `wholesale-application-form` | New | `heading`, `description`, `success_message`, `submit_label` | None | `button` |
| `quick-order-form` | New | `heading`, `collection`, `products_per_page` | None | `button`, `quantity-selector`, `wholesale-badge` |
| `wholesale-cta-banner` | New | `heading`, `subtitle`, `button_label`, `button_url`, `show_to_logged_in` | None | `button` |

---

## Snippets

| Snippet | New/Modified | Parameters | Used By |
|---|---|---|---|
| `price` | Modified | + `show_wholesale` (boolean), + `customer_tags` (string) | `product-card`, `product-detail` |
| `product-card` | Modified | (existing params) | `product-grid`, `featured-products`, `related-products` |
| `wholesale-price` | New | `product` (product), `customer_tags` (string) | `product-detail` |
| `wholesale-badge` | New | `tier` (string), `label` (string) | `product-card`, `product-detail`, `wholesale-pricing-table`, `quick-order-form` |
| `wholesale-cart-summary` | New | `cart` (cart), `customer_tags` (string) | `cart-content` |
| `quantity-selector` | Modified | + `min` (number) — allow custom minimum | `product-detail`, `quick-order-form` |

---

## Build Order

| # | File | Type | Reason |
|---|---|---|---|
| 1 | `config/settings_schema.json` | Modified | Add wholesale settings group (tier config, enable/disable) |
| 2 | `locales/en.default.json` | Modified | Add all `wholesale.*` translation keys |
| 3 | `locales/en.default.schema.json` | Modified | Add editor labels for wholesale settings and sections |
| 4 | `assets/critical.css` | Modified | Add shared wholesale CSS (badge variant, pricing table base) |
| 5 | `snippets/wholesale-badge.liquid` | New | Shared wholesale tier badge component |
| 6 | `snippets/wholesale-price.liquid` | New | Wholesale price display with savings calculation |
| 7 | `snippets/wholesale-cart-summary.liquid` | New | Cart savings summary for wholesale customers |
| 8 | `snippets/price.liquid` | Modified | Add wholesale price branch using customer tags |
| 9 | `snippets/product-card.liquid` | Modified | Show wholesale badge and price for tagged customers |
| 10 | `snippets/quantity-selector.liquid` | Modified | Support custom `min` parameter for minimum order quantities |
| 11 | `sections/product-detail.liquid` | Modified | Show wholesale pricing, enforce min quantity, show tier badge |
| 12 | `sections/wholesale-pricing-table.liquid` | New | Volume discount / quantity break table |
| 13 | `sections/cart-content.liquid` | Modified | Show wholesale savings summary, validate min quantities |
| 14 | `sections/wholesale-application-form.liquid` | New | Wholesale account application form |
| 15 | `sections/quick-order-form.liquid` | New | Bulk order form for wholesale customers |
| 16 | `sections/wholesale-cta-banner.liquid` | New | "Apply for wholesale" CTA for non-wholesale customers |
| 17 | `templates/product.json` | Modified | Add `wholesale-pricing-table` section |
| 18 | `templates/page.wholesale-application.json` | New | Compose wholesale application page |
| 19 | `templates/page.quick-order.json` | New | Compose quick order form page |

---

## Detailed File Specifications

### 1. `config/settings_schema.json` (Modified)

Add a new settings group after the existing "Cart" group:

```json
{
  "name": "t:settings.wholesale.name",
  "settings": [
    {
      "id": "wholesale_enabled",
      "type": "checkbox",
      "label": "t:settings.wholesale.enabled.label",
      "info": "t:settings.wholesale.enabled.info",
      "default": true
    },
    {
      "type": "header",
      "content": "t:settings.wholesale.tier_1_header"
    },
    {
      "id": "wholesale_tier_1_tag",
      "type": "text",
      "label": "t:settings.wholesale.tier_1_tag.label",
      "default": "wholesale"
    },
    {
      "id": "wholesale_tier_1_name",
      "type": "text",
      "label": "t:settings.wholesale.tier_1_name.label",
      "default": "Wholesale"
    },
    {
      "id": "wholesale_tier_1_discount",
      "type": "number",
      "label": "t:settings.wholesale.tier_1_discount.label",
      "default": 15
    },
    {
      "id": "wholesale_tier_1_min_qty",
      "type": "number",
      "label": "t:settings.wholesale.tier_1_min_qty.label",
      "default": 10
    },
    {
      "type": "header",
      "content": "t:settings.wholesale.tier_2_header"
    },
    {
      "id": "wholesale_tier_2_tag",
      "type": "text",
      "label": "t:settings.wholesale.tier_2_tag.label",
      "default": "vip"
    },
    {
      "id": "wholesale_tier_2_name",
      "type": "text",
      "label": "t:settings.wholesale.tier_2_name.label",
      "default": "VIP"
    },
    {
      "id": "wholesale_tier_2_discount",
      "type": "number",
      "label": "t:settings.wholesale.tier_2_discount.label",
      "default": 20
    },
    {
      "id": "wholesale_tier_2_min_qty",
      "type": "number",
      "label": "t:settings.wholesale.tier_2_min_qty.label",
      "default": 5
    },
    {
      "type": "header",
      "content": "t:settings.wholesale.tier_3_header"
    },
    {
      "id": "wholesale_tier_3_tag",
      "type": "text",
      "label": "t:settings.wholesale.tier_3_tag.label",
      "default": "distributor"
    },
    {
      "id": "wholesale_tier_3_name",
      "type": "text",
      "label": "t:settings.wholesale.tier_3_name.label",
      "default": "Distributor"
    },
    {
      "id": "wholesale_tier_3_discount",
      "type": "number",
      "label": "t:settings.wholesale.tier_3_discount.label",
      "default": 25
    },
    {
      "id": "wholesale_tier_3_min_qty",
      "type": "number",
      "label": "t:settings.wholesale.tier_3_min_qty.label",
      "default": 25
    },
    {
      "type": "header",
      "content": "t:settings.wholesale.shipping_header"
    },
    {
      "id": "wholesale_free_shipping_threshold",
      "type": "number",
      "label": "t:settings.wholesale.free_shipping_threshold.label",
      "default": 500
    }
  ]
}
```

---

### 2. `snippets/wholesale-badge.liquid` (New)

```
{% doc %}
  Renders a wholesale tier badge.

  @param {string} tier  - Tier identifier: "wholesale", "vip", or "distributor". Required.
  @param {string} label - Badge text to display. Required.

  @example
  {% assign ws_label = 'wholesale.badge_wholesale' | t %}
  {% render 'wholesale-badge', tier: 'wholesale', label: ws_label %}
{% enddoc %}
```

**Behavior:**
- Renders a `<span>` with class `wholesale-badge wholesale-badge--{{ tier }}`
- Each tier gets a distinct color:
  - `wholesale` -- `--color-primary` background
  - `vip` -- `--color-success` background
  - `distributor` -- dark background (`--color-text`)

**CSS:** Raw `<style>` tag (snippet rule). Styles for `.wholesale-badge` and tier modifiers.

---

### 3. `snippets/wholesale-price.liquid` (New)

```
{% doc %}
  Renders the wholesale price display for a product, showing the tier price,
  retail comparison, and savings percentage.

  @param {product} product        - Shopify product object. Required.
  @param {string}  customer_tags  - Comma-separated customer tags string. Required.

  @example
  {% assign tags_string = customer.tags | join: ',' %}
  {% render 'wholesale-price', product: product, customer_tags: tags_string %}
{% enddoc %}
```

**Logic:**
1. Split `customer_tags` by comma into an array
2. Check which tier tag is present (priority: distributor > vip > wholesale)
3. Look for product metafield `product.metafields.custom.wholesale_price` first
4. If no metafield, calculate from the global tier discount percentage in settings
5. Display:
   - Wholesale price (large, bold)
   - Retail price (strikethrough)
   - "You save X%" badge

**Important:** Translations must be pre-assigned to variables before passing to `{% render %}`. The snippet receives `customer_tags` as a string (not the customer object, since snippets have isolated scope).

---

### 4. `snippets/wholesale-cart-summary.liquid` (New)

```
{% doc %}
  Renders a wholesale savings summary in the cart, showing per-item and total
  savings compared to retail pricing.

  @param {cart}   cart           - Shopify cart object. Required.
  @param {string} customer_tags  - Comma-separated customer tags string. Required.

  @example
  {% assign tags_string = customer.tags | join: ',' %}
  {% render 'wholesale-cart-summary', cart: cart, customer_tags: tags_string %}
{% enddoc %}
```

**Display:**
- A summary box above the cart totals showing:
  - "Wholesale pricing applied" heading with tier badge
  - "You are saving $XX.XX on this order" line
  - Per-item breakdown (optional, collapsible via `<details>`)

**Note:** The savings shown are estimated based on metafield/settings prices. Actual checkout price depends on Shopify discount configuration.

---

### 5. `snippets/price.liquid` (Modified)

**Changes:**
- Add two new optional parameters: `show_wholesale` (boolean), `customer_tags` (string)
- When `show_wholesale` is true and `customer_tags` contains a wholesale tier tag:
  - Show the wholesale price (from metafield or calculated from discount percentage)
  - Show the retail price with strikethrough
- When `show_wholesale` is false or customer has no wholesale tag: render exactly as today (no behavior change)

**Backward compatibility:** All existing callers pass no `show_wholesale` parameter, so they default to `false` and behavior is unchanged.

---

### 6. `snippets/product-card.liquid` (Modified)

**Changes:**
- Before the badge area, check `settings.wholesale_enabled`
- If wholesale is enabled, pass customer tags to the price snippet:
  ```liquid
  {% if settings.wholesale_enabled %}
    {% if customer %}
      {% assign tags_string = customer.tags | join: ',' %}
    {% endif %}
  {% endif %}
  ```
- In the badge area, if customer has a wholesale tag, render `wholesale-badge` instead of (or alongside) the bestseller badge
- In the price area, pass `show_wholesale: true` and `customer_tags: tags_string` to the `price` snippet

**Important note on scope:** The `customer` object is available globally in Liquid, but inside a snippet rendered with `{% render %}`, it is NOT available. The parent section must pass customer tag data as a parameter. However, `settings` (global theme settings) ARE available in snippets.

**Revised approach:** Since `product-card` is rendered via `{% render %}`, the calling section must pass a `customer_tags` parameter. Modify all callers:
- `sections/product-grid.liquid` — add `customer_tags` pass-through
- `sections/featured-products.liquid` — add `customer_tags` pass-through
- `sections/related-products.liquid` — add `customer_tags` pass-through

Each caller adds before its `{% for %}` loop:
```liquid
{% assign ws_tags = '' %}
{% if customer %}
  {% assign ws_tags = customer.tags | join: ',' %}
{% endif %}
```

Then passes it:
```liquid
{% render 'product-card', product: product, customer_tags: ws_tags %}
```

---

### 7. `snippets/quantity-selector.liquid` (Modified)

**Changes:**
- Add optional `min` parameter (currently hardcoded to `min="1"`)
- Replace `min="1"` with `min="{{ min_value }}"` where `min_value` defaults to 1
- Add at the top: `{% assign min_value = min | default: 1 %}`
- Update the `clamp` function in the `<script>` to read from the input's `min` attribute (it already does this: `parseInt(input.min, 10) || 1`)

---

### 8. `sections/wholesale-pricing-table.liquid` (New)

**Purpose:** Displays a volume discount / quantity break table on the product page.

**Schema:**
```json
{
  "name": "t:sections.wholesale_pricing_table.name",
  "settings": [
    {
      "type": "text",
      "id": "heading",
      "label": "t:sections.wholesale_pricing_table.settings.heading.label",
      "default": "Volume pricing"
    },
    {
      "type": "checkbox",
      "id": "show_for_all",
      "label": "t:sections.wholesale_pricing_table.settings.show_for_all.label",
      "info": "t:sections.wholesale_pricing_table.settings.show_for_all.info",
      "default": false
    }
  ],
  "blocks": [
    {
      "type": "quantity_break",
      "name": "t:blocks.quantity_break.name",
      "settings": [
        {
          "type": "number",
          "id": "min_quantity",
          "label": "t:blocks.quantity_break.settings.min_quantity.label",
          "default": 10
        },
        {
          "type": "number",
          "id": "discount_percent",
          "label": "t:blocks.quantity_break.settings.discount_percent.label",
          "default": 15
        },
        {
          "type": "text",
          "id": "label",
          "label": "t:blocks.quantity_break.settings.label.label",
          "default": "Wholesale"
        }
      ]
    }
  ],
  "presets": [
    {
      "name": "Wholesale pricing table",
      "blocks": [
        { "type": "quantity_break", "settings": { "min_quantity": 10, "discount_percent": 15, "label": "Wholesale" } },
        { "type": "quantity_break", "settings": { "min_quantity": 25, "discount_percent": 20, "label": "Bulk" } },
        { "type": "quantity_break", "settings": { "min_quantity": 50, "discount_percent": 25, "label": "Distributor" } }
      ]
    }
  ]
}
```

**Rendering logic:**
1. If `show_for_all` is false, only show when `customer.tags` contains a wholesale-tier tag
2. Renders a `<table>` with columns: Quantity, Discount, Unit Price (calculated from `product.price`)
3. Highlights the row matching the customer's current tier
4. Uses `{% stylesheet %}` for table styling (section, not snippet)

**HTML structure:**
```html
<section class="wholesale-pricing-table section-padding">
  <div class="container">
    <h2>{{ section.settings.heading }}</h2>
    <table>
      <thead>
        <tr>
          <th>{{ 'wholesale.table_quantity' | t }}</th>
          <th>{{ 'wholesale.table_discount' | t }}</th>
          <th>{{ 'wholesale.table_unit_price' | t }}</th>
        </tr>
      </thead>
      <tbody>
        {% for block in section.blocks %}
          <!-- Calculate discounted price from product.price -->
          <tr>
            <td>{{ block.settings.min_quantity }}+</td>
            <td>{{ block.settings.discount_percent }}%</td>
            <td>{{ discounted_price | money }}</td>
          </tr>
        {% endfor %}
      </tbody>
    </table>
  </div>
</section>
```

---

### 9. `sections/product-detail.liquid` (Modified)

**Changes to schema:**
- Add `show_wholesale_pricing` checkbox setting (default: true)

**Changes to template:**
- After the price area, add a wholesale price display block:

```liquid
{% if settings.wholesale_enabled %}
  {% if section.settings.show_wholesale_pricing %}
    {% if customer %}
      {% assign ws_tags = customer.tags | join: ',' %}
      {% render 'wholesale-price', product: product, customer_tags: ws_tags %}
    {% endif %}
  {% endif %}
{% endif %}
```

- Modify the quantity selector to enforce minimum order quantity for wholesale customers:

```liquid
{% assign ws_min = 1 %}
{% if settings.wholesale_enabled %}
  {% if customer %}
    {% if customer.tags contains settings.wholesale_tier_3_tag %}
      {% assign ws_min = settings.wholesale_tier_3_min_qty %}
    {% elsif customer.tags contains settings.wholesale_tier_2_tag %}
      {% assign ws_min = settings.wholesale_tier_2_min_qty %}
    {% elsif customer.tags contains settings.wholesale_tier_1_tag %}
      {% assign ws_min = settings.wholesale_tier_1_min_qty %}
    {% endif %}
  {% endif %}
{% endif %}

{% render 'quantity-selector', value: ws_min, name: 'quantity', min: ws_min %}
```

- Add a "Minimum order: X units" notice below the quantity selector for wholesale customers

---

### 10. `sections/cart-content.liquid` (Modified)

**Changes to schema:**
- Add `show_wholesale_savings` checkbox setting (default: true)

**Changes to template:**
- Before the totals section, insert wholesale savings summary:

```liquid
{% if settings.wholesale_enabled %}
  {% if section.settings.show_wholesale_savings %}
    {% if customer %}
      {% assign ws_tags = customer.tags | join: ',' %}
      {% render 'wholesale-cart-summary', cart: cart, customer_tags: ws_tags %}
    {% endif %}
  {% endif %}
{% endif %}
```

- Add JavaScript for minimum quantity validation:
  - On quantity change, check if new quantity is below the wholesale minimum
  - Show a warning message: "Minimum order quantity for wholesale is X units"
  - Prevent decrementing below the minimum

---

### 11. `sections/wholesale-application-form.liquid` (New)

**Purpose:** A page-level section that renders a wholesale account application form.

**Schema:**
```json
{
  "name": "t:sections.wholesale_application.name",
  "settings": [
    {
      "type": "text",
      "id": "heading",
      "label": "t:sections.wholesale_application.settings.heading.label",
      "default": "Apply for wholesale pricing"
    },
    {
      "type": "richtext",
      "id": "description",
      "label": "t:sections.wholesale_application.settings.description.label"
    },
    {
      "type": "text",
      "id": "success_message",
      "label": "t:sections.wholesale_application.settings.success_message.label",
      "default": "Thank you! Your application has been submitted."
    },
    {
      "type": "text",
      "id": "submit_label",
      "label": "t:sections.wholesale_application.settings.submit_label.label",
      "default": "Submit application"
    },
    {
      "type": "text",
      "id": "notification_email",
      "label": "t:sections.wholesale_application.settings.notification_email.label",
      "info": "t:sections.wholesale_application.settings.notification_email.info"
    }
  ],
  "presets": [
    {
      "name": "Wholesale application"
    }
  ]
}
```

**Form fields** (hardcoded in template, all use translation keys):
- Business name (text, required)
- Contact name (text, required)
- Email (email, required)
- Phone (tel, optional)
- Tax ID / Business number (text, required)
- Business type (select: retailer, restaurant/bar, distributor, other)
- Expected monthly order volume (select: ranges)
- Website URL (url, optional)
- Additional notes (textarea, optional)

**Form submission approach:**
- Uses a Shopify `contact` form (`{% form 'contact' %}`) which sends data to the store's notification email
- The form `tags` the submission with "wholesale-application" for filtering
- JavaScript provides client-side validation and a success state
- The merchant manually reviews applications and tags the customer account with the appropriate tier tag

**HTML structure:**
```html
<section class="wholesale-application section-padding">
  <div class="container">
    <div class="wholesale-application__content">
      <h1>{{ section.settings.heading }}</h1>
      {% if section.settings.description != blank %}
        <div class="wholesale-application__description">
          {{ section.settings.description }}
        </div>
      {% endif %}

      {% form 'contact' %}
        <!-- Hidden fields for Shopify contact form routing -->
        <input type="hidden" name="contact[tags]" value="wholesale-application">
        <input type="hidden" name="contact[subject]" value="{{ 'wholesale.application_subject' | t }}">

        <!-- Visible form fields -->
        <div class="wholesale-application__fields">
          <!-- ... field groups ... -->
        </div>

        <button type="submit" class="btn btn--primary btn--full-width">
          {{ section.settings.submit_label }}
        </button>
      {% endform %}
    </div>
  </div>
</section>
```

---

### 12. `sections/quick-order-form.liquid` (New)

**Purpose:** Bulk order form where wholesale customers add multiple products/variants at once.

**Schema:**
```json
{
  "name": "t:sections.quick_order.name",
  "settings": [
    {
      "type": "text",
      "id": "heading",
      "label": "t:sections.quick_order.settings.heading.label",
      "default": "Quick order"
    },
    {
      "type": "collection",
      "id": "collection",
      "label": "t:sections.quick_order.settings.collection.label"
    },
    {
      "type": "range",
      "id": "products_per_page",
      "label": "t:sections.quick_order.settings.products_per_page.label",
      "min": 10,
      "max": 50,
      "step": 5,
      "default": 20
    }
  ],
  "presets": [
    {
      "name": "Quick order form"
    }
  ]
}
```

**Rendering logic:**
1. Gate behind wholesale customer check:
   ```liquid
   {% if customer %}
     {% if customer.tags contains settings.wholesale_tier_1_tag %}
       {% assign is_wholesale = true %}
     {% endif %}
     {% if customer.tags contains settings.wholesale_tier_2_tag %}
       {% assign is_wholesale = true %}
     {% endif %}
     {% if customer.tags contains settings.wholesale_tier_3_tag %}
       {% assign is_wholesale = true %}
     {% endif %}
   {% endif %}
   ```
2. If not wholesale, show a message with link to login or apply
3. If wholesale, render the collection's products in a table format:

**HTML structure:**
```html
<section class="quick-order section-padding">
  <div class="container">
    <h1>{{ section.settings.heading }}</h1>

    {% if is_wholesale %}
      <div class="quick-order__toolbar">
        <input type="search" placeholder="{{ 'wholesale.search_products' | t }}"
               data-quick-order-search aria-label="{{ 'wholesale.search_products' | t }}">
        <button class="btn btn--primary" data-quick-order-submit disabled>
          {{ 'wholesale.add_all_to_cart' | t }}
          (<span data-quick-order-count>0</span>)
        </button>
      </div>

      <table class="quick-order__table">
        <thead>
          <tr>
            <th>{{ 'wholesale.table_product' | t }}</th>
            <th>{{ 'wholesale.table_sku' | t }}</th>
            <th>{{ 'wholesale.table_price' | t }}</th>
            <th>{{ 'wholesale.table_stock' | t }}</th>
            <th>{{ 'wholesale.table_quantity' | t }}</th>
          </tr>
        </thead>
        <tbody>
          {% for product in collection.products %}
            {% for variant in product.variants %}
              <tr data-quick-order-row data-variant-id="{{ variant.id }}"
                  data-product-title="{{ product.title | escape }}"
                  data-variant-title="{{ variant.title | escape }}">
                <td>
                  {{ product.title }}
                  {% if variant.title != 'Default Title' %}
                    <span class="quick-order__variant-label">{{ variant.title }}</span>
                  {% endif %}
                </td>
                <td>{{ variant.sku }}</td>
                <td>{{ variant.price | money }}</td>
                <td>
                  {% if variant.available %}
                    {% if variant.inventory_quantity > 0 %}
                      {{ variant.inventory_quantity }}
                    {% else %}
                      {{ 'products.in_stock' | t }}
                    {% endif %}
                  {% else %}
                    {{ 'products.out_of_stock' | t }}
                  {% endif %}
                </td>
                <td>
                  {% if variant.available %}
                    <input type="number" min="0" max="999" value="0" step="1"
                           class="quick-order__qty-input"
                           data-variant-id="{{ variant.id }}"
                           aria-label="{{ 'wholesale.quantity_for' | t: title: product.title }}">
                  {% else %}
                    --
                  {% endif %}
                </td>
              </tr>
            {% endfor %}
          {% endfor %}
        </tbody>
      </table>
    {% else %}
      <div class="quick-order__gated">
        <p>{{ 'wholesale.quick_order_gated' | t }}</p>
        {% assign login_label = 'customer.submit_login' | t %}
        {% render 'button', label: login_label, url: routes.account_login_url, variant: 'primary' %}
      </div>
    {% endif %}
  </div>
</section>
```

**JavaScript (in `<script>` tag):**
- Client-side search filtering: filter table rows by product title as user types
- Track quantity changes: count items with quantity > 0, update the "Add all to cart" button count
- "Add all to cart" button handler:
  1. Collect all rows with quantity > 0
  2. Build an array of `{ id: variant_id, quantity: qty }` objects
  3. POST each item to `/cart/add.js` sequentially (Shopify does not support batch add)
  4. After all items added, redirect to `/cart` or show success toast
  5. Disable button during submission, show loading state

```javascript
// Pseudocode for the add-all handler
var items = [];
document.querySelectorAll('[data-quick-order-row]').forEach(function(row) {
  var input = row.querySelector('.quick-order__qty-input');
  if (input && parseInt(input.value) > 0) {
    items.push({ id: parseInt(row.dataset.variantId), quantity: parseInt(input.value) });
  }
});

// Sequential add (Shopify /cart/add.js does not support arrays in all themes)
function addNext(index) {
  if (index >= items.length) {
    window.location.href = '/cart';
    return;
  }
  fetch('/cart/add.js', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ items: [items[index]] })
  }).then(function() { addNext(index + 1); });
}
addNext(0);
```

**Note:** Shopify's `/cart/add.js` endpoint DOES support an `items` array for batch adding. The preferred approach:
```javascript
fetch('/cart/add.js', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ items: items })
});
```

---

### 13. `sections/wholesale-cta-banner.liquid` (New)

**Purpose:** A banner section encouraging non-wholesale customers to apply for wholesale pricing. Can be added to the homepage or any page via the theme editor.

**Schema:**
```json
{
  "name": "t:sections.wholesale_cta.name",
  "settings": [
    {
      "type": "text",
      "id": "heading",
      "label": "t:sections.wholesale_cta.settings.heading.label",
      "default": "Save more with wholesale pricing"
    },
    {
      "type": "textarea",
      "id": "subtitle",
      "label": "t:sections.wholesale_cta.settings.subtitle.label",
      "default": "Get exclusive pricing on bulk orders. Apply for a wholesale account today."
    },
    {
      "type": "text",
      "id": "button_label",
      "label": "t:sections.wholesale_cta.settings.button_label.label",
      "default": "Apply for wholesale"
    },
    {
      "type": "url",
      "id": "button_url",
      "label": "t:sections.wholesale_cta.settings.button_url.label"
    }
  ],
  "presets": [
    {
      "name": "Wholesale CTA banner"
    }
  ]
}
```

**Rendering logic:**
- If `settings.wholesale_enabled` is false, render nothing
- If customer is logged in AND has a wholesale tag, render nothing (they already have wholesale)
- Otherwise, show the banner with heading, subtitle, and CTA button

---

### 14. `templates/product.json` (Modified)

Add the `wholesale-pricing-table` section between `main` and `tabs`:

```json
{
  "sections": {
    "main": { "type": "product-detail", "settings": {} },
    "wholesale_pricing": {
      "type": "wholesale-pricing-table",
      "settings": { "heading": "Volume pricing" },
      "blocks": {
        "break_1": { "type": "quantity_break", "settings": { "min_quantity": 10, "discount_percent": 15, "label": "Wholesale" } },
        "break_2": { "type": "quantity_break", "settings": { "min_quantity": 25, "discount_percent": 20, "label": "Bulk" } },
        "break_3": { "type": "quantity_break", "settings": { "min_quantity": 50, "discount_percent": 25, "label": "Distributor" } }
      },
      "block_order": ["break_1", "break_2", "break_3"]
    },
    "tabs": { "type": "product-tabs", "settings": {} },
    "related": { "type": "related-products", "settings": {} }
  },
  "order": ["main", "wholesale_pricing", "tabs", "related"]
}
```

---

### 15. `templates/page.wholesale-application.json` (New)

```json
{
  "sections": {
    "main": {
      "type": "wholesale-application-form",
      "settings": {
        "heading": "Apply for wholesale pricing",
        "submit_label": "Submit application"
      }
    }
  },
  "order": ["main"]
}
```

**Merchant setup:** Create a page in Shopify admin with handle `wholesale-application` and assign this template.

---

### 16. `templates/page.quick-order.json` (New)

```json
{
  "sections": {
    "main": {
      "type": "quick-order-form",
      "settings": {
        "heading": "Quick order",
        "products_per_page": 20
      }
    }
  },
  "order": ["main"]
}
```

**Merchant setup:** Create a page in Shopify admin with handle `quick-order` and assign this template.

---

## Metafield Requirements

These metafields must be defined in the Shopify admin under Settings > Custom data > Products:

| Namespace | Key | Type | Description |
|---|---|---|---|
| `custom` | `wholesale_price` | `number_decimal` (money) | Per-product wholesale price override. When set, this takes priority over the global tier discount percentage. |
| `custom` | `tier_2_price` | `number_decimal` (money) | Per-product VIP tier price override. |
| `custom` | `tier_3_price` | `number_decimal` (money) | Per-product distributor tier price override. |
| `custom` | `wholesale_min_qty` | `number_integer` | Per-product minimum order quantity override. Takes priority over the global tier minimum. |
| `custom` | `volume_pricing_enabled` | `boolean` | Enable/disable volume pricing table for this specific product. |

**Access in Liquid:**
```liquid
{{ product.metafields.custom.wholesale_price.value }}
{{ product.metafields.custom.tier_2_price.value }}
{{ product.metafields.custom.tier_3_price.value }}
{{ product.metafields.custom.wholesale_min_qty.value }}
{{ product.metafields.custom.volume_pricing_enabled.value }}
```

**Metafield definition setup (Admin API or Shopify admin):**
- Namespace: `custom`
- Pin to product listing for Liquid access (metafields are only accessible in Liquid when they have a definition)
- For money types, use `number_decimal` and format with `| money` filter in Liquid

---

## Translation Keys

### `locales/en.default.json` additions

```json
{
  "wholesale": {
    "badge_wholesale": "Wholesale",
    "badge_vip": "VIP",
    "badge_distributor": "Distributor",
    "your_price": "Your price",
    "retail_price": "Retail price",
    "you_save": "You save {{ percent }}%",
    "you_save_amount": "You save {{ amount }}",
    "min_order_notice": "Minimum order: {{ quantity }} units",
    "table_quantity": "Quantity",
    "table_discount": "Discount",
    "table_unit_price": "Unit price",
    "table_product": "Product",
    "table_sku": "SKU",
    "table_price": "Price",
    "table_stock": "Stock",
    "volume_pricing_heading": "Volume pricing",
    "savings_heading": "Wholesale savings",
    "savings_total": "Total savings on this order",
    "savings_per_item": "{{ title }}: saving {{ amount }}",
    "pricing_applied": "Wholesale pricing applied",
    "pricing_note": "Prices shown reflect your wholesale tier. Final pricing is applied at checkout.",
    "add_all_to_cart": "Add all to cart",
    "search_products": "Search products...",
    "quantity_for": "Quantity for {{ title }}",
    "quick_order_gated": "The quick order form is available for wholesale customers. Please log in or apply for a wholesale account.",
    "quick_order_adding": "Adding items to cart...",
    "quick_order_success": "{{ count }} items added to cart",
    "quick_order_error": "Some items could not be added. Please try again.",
    "application_heading": "Apply for wholesale pricing",
    "application_description": "Fill out the form below to apply for a wholesale account. We will review your application and get back to you within 2 business days.",
    "application_subject": "Wholesale account application",
    "application_success": "Thank you! Your wholesale application has been submitted. We will review it and contact you shortly.",
    "field_business_name": "Business name",
    "field_contact_name": "Contact name",
    "field_email": "Email address",
    "field_phone": "Phone number",
    "field_tax_id": "Tax ID / Business number",
    "field_business_type": "Business type",
    "field_business_type_retailer": "Retailer",
    "field_business_type_restaurant": "Restaurant / Bar",
    "field_business_type_distributor": "Distributor",
    "field_business_type_other": "Other",
    "field_order_volume": "Expected monthly order volume",
    "field_volume_small": "Under $500",
    "field_volume_medium": "$500 - $2,000",
    "field_volume_large": "$2,000 - $5,000",
    "field_volume_enterprise": "Over $5,000",
    "field_website": "Website URL",
    "field_notes": "Additional notes",
    "field_required": "Required",
    "cta_heading": "Save more with wholesale pricing",
    "cta_subtitle": "Get exclusive pricing on bulk orders. Apply for a wholesale account today.",
    "cta_button": "Apply for wholesale",
    "min_qty_warning": "Minimum quantity for this item is {{ quantity }}"
  }
}
```

### `locales/en.default.schema.json` additions

```json
{
  "settings": {
    "wholesale": {
      "name": "Wholesale",
      "enabled": {
        "label": "Enable wholesale pricing",
        "info": "Show wholesale prices to customers with wholesale tags"
      },
      "tier_1_header": "Tier 1",
      "tier_1_tag": { "label": "Customer tag" },
      "tier_1_name": { "label": "Tier name" },
      "tier_1_discount": { "label": "Discount percentage" },
      "tier_1_min_qty": { "label": "Minimum order quantity" },
      "tier_2_header": "Tier 2",
      "tier_2_tag": { "label": "Customer tag" },
      "tier_2_name": { "label": "Tier name" },
      "tier_2_discount": { "label": "Discount percentage" },
      "tier_2_min_qty": { "label": "Minimum order quantity" },
      "tier_3_header": "Tier 3",
      "tier_3_tag": { "label": "Customer tag" },
      "tier_3_name": { "label": "Tier name" },
      "tier_3_discount": { "label": "Discount percentage" },
      "tier_3_min_qty": { "label": "Minimum order quantity" },
      "shipping_header": "Wholesale shipping",
      "free_shipping_threshold": { "label": "Free shipping threshold (wholesale)" }
    }
  },
  "sections": {
    "wholesale_pricing_table": {
      "name": "Wholesale pricing table",
      "settings": {
        "heading": { "label": "Heading" },
        "show_for_all": {
          "label": "Show to all customers",
          "info": "When disabled, only wholesale-tagged customers see this table"
        }
      }
    },
    "wholesale_application": {
      "name": "Wholesale application",
      "settings": {
        "heading": { "label": "Heading" },
        "description": { "label": "Description" },
        "success_message": { "label": "Success message" },
        "submit_label": { "label": "Submit button label" },
        "notification_email": {
          "label": "Notification email",
          "info": "Email address for application notifications (uses store email if empty)"
        }
      }
    },
    "quick_order": {
      "name": "Quick order form",
      "settings": {
        "heading": { "label": "Heading" },
        "collection": { "label": "Collection" },
        "products_per_page": { "label": "Products per page" }
      }
    },
    "wholesale_cta": {
      "name": "Wholesale CTA banner",
      "settings": {
        "heading": { "label": "Heading" },
        "subtitle": { "label": "Subtitle" },
        "button_label": { "label": "Button label" },
        "button_url": { "label": "Button URL" }
      }
    }
  },
  "blocks": {
    "quantity_break": {
      "name": "Quantity break",
      "settings": {
        "min_quantity": { "label": "Minimum quantity" },
        "discount_percent": { "label": "Discount percentage" },
        "label": { "label": "Tier label" }
      }
    }
  }
}
```

---

## JavaScript Approach

### Vanilla JS only (no frameworks)

All JavaScript uses `<script>` tags (NOT `{% javascript %}`), per the project coding conventions.

### 1. Product page — wholesale pricing display

**File:** `sections/product-detail.liquid` (inline `<script>`)

- No additional JS needed for basic wholesale price display (handled by Liquid)
- Minimum quantity enforcement: update quantity selector minimum via `data-wholesale-min` attribute
- When user tries to set quantity below minimum, snap back to minimum and show a warning message

### 2. Quick order form — bulk add to cart

**File:** `sections/quick-order-form.liquid` (inline `<script>`)

Functions:
- `filterProducts(query)` — hide table rows that do not match the search input
- `updateItemCount()` — count rows with quantity > 0 and update the button badge
- `addAllToCart()` — collect all variant IDs with quantities, POST to `/cart/add.js` with `items` array, then redirect to `/cart`

Event listeners:
- `input` on search field -> `filterProducts()`
- `input` on quantity inputs -> `updateItemCount()`
- `click` on "Add all to cart" button -> `addAllToCart()`

### 3. Cart page — minimum quantity validation

**File:** `sections/cart-content.liquid` (inline `<script>`, extend existing)

- On `quantity:change` event, check if the customer has a wholesale tag (passed via `data-wholesale-min` on the quantity selector)
- If new quantity < minimum, revert to minimum and show warning
- This is a UX-layer check only; actual enforcement must happen via Shopify Scripts or checkout validation

### 4. Wholesale application form — client-side validation

**File:** `sections/wholesale-application-form.liquid` (inline `<script>`)

- Validate required fields on submit
- Show inline error messages
- On successful submission (Shopify contact form), show success message and hide form

---

## Shopify Limitations and Workarounds

### Price enforcement at checkout

Liquid can only change the DISPLAY of prices. It cannot alter the actual checkout price. For actual wholesale pricing enforcement, the merchant must use one of:

1. **Shopify Automatic Discounts** — Create automatic discounts based on customer segments (available on all plans)
2. **Shopify Scripts** (Shopify Plus only) — Line item scripts that apply percentage discounts based on customer tags
3. **Discount Codes** — Generate unique discount codes per wholesale tier and share with wholesale customers
4. **Shopify B2B** (Shopify Plus only) — Native B2B features with company profiles, catalogs, and net payment terms

The theme displays a note: "Prices shown reflect your wholesale tier. Final pricing is applied at checkout." This sets the right expectation.

### Customer tag assignment

Customer tags cannot be set by Liquid or front-end JavaScript. The wholesale application form submits via the Shopify contact form. The merchant must then:

1. Review the application in Shopify admin (Settings > Notifications or via email)
2. Navigate to the customer's account in Shopify admin
3. Add the appropriate tier tag (`wholesale`, `vip`, or `distributor`)
4. Optionally use Shopify Flow to automate tagging based on form submissions

### Metafield availability

Product metafields are only accessible in Liquid when they have a metafield definition created in Shopify admin. The implementation plan relies on merchants creating these definitions before the wholesale price display works with per-product overrides.

---

## CSS Approach

### Shared styles in `critical.css`

Add to `critical.css`:
- `.wholesale-badge` base styles and tier modifier classes
- `.wholesale-table` base styles for the pricing table

### Component-specific styles

- Sections use `{% stylesheet %}` blocks
- Snippets use raw `<style>` tags
- No external CSS libraries

### Design alignment

Wholesale elements use the existing BrewMaster design language:
- Amber primary (`#d97706`) for wholesale highlights and badges
- Same `--border-radius`, `--color-border`, `--color-background-alt` variables
- Same typographic scale
- Same `.btn` classes for buttons

---

## Testing Checklist

- [ ] Non-logged-in visitor: sees retail prices only, no wholesale badges, no quick order access
- [ ] Logged-in customer without wholesale tag: sees retail prices, sees wholesale CTA banner, can access application form
- [ ] Logged-in customer with `wholesale` tag: sees wholesale prices, savings, minimum quantity enforced, can access quick order form
- [ ] Logged-in customer with `vip` tag: sees VIP prices (higher discount), lower minimum
- [ ] Logged-in customer with `distributor` tag: sees distributor prices (highest discount), highest minimum
- [ ] Product with `wholesale_price` metafield: metafield price takes priority over calculated percentage
- [ ] Product without metafield: calculated from global tier discount percentage
- [ ] Quick order form: search filters correctly, quantity tracking works, batch add to cart succeeds
- [ ] Cart: savings summary displays correctly, minimum quantity validation works
- [ ] Wholesale application: form submits via contact form, success message shows
- [ ] Mobile responsive: all wholesale elements work on mobile viewports
- [ ] Accessibility: all form fields have labels, keyboard navigation works, screen reader announces wholesale prices

---

## File Summary

### New files (10)

| File | Type |
|---|---|
| `snippets/wholesale-badge.liquid` | Snippet |
| `snippets/wholesale-price.liquid` | Snippet |
| `snippets/wholesale-cart-summary.liquid` | Snippet |
| `sections/wholesale-pricing-table.liquid` | Section |
| `sections/wholesale-application-form.liquid` | Section |
| `sections/quick-order-form.liquid` | Section |
| `sections/wholesale-cta-banner.liquid` | Section |
| `templates/page.wholesale-application.json` | Template |
| `templates/page.quick-order.json` | Template |
| (none — no new blocks directory files needed) | |

### Modified files (9)

| File | Changes |
|---|---|
| `config/settings_schema.json` | Add wholesale settings group |
| `locales/en.default.json` | Add `wholesale.*` keys |
| `locales/en.default.schema.json` | Add wholesale section/settings labels |
| `assets/critical.css` | Add shared wholesale badge and table styles |
| `snippets/price.liquid` | Add wholesale price branch |
| `snippets/product-card.liquid` | Add wholesale badge and price display |
| `snippets/quantity-selector.liquid` | Add `min` parameter support |
| `sections/product-detail.liquid` | Add wholesale pricing, min qty enforcement |
| `sections/cart-content.liquid` | Add wholesale savings summary |

### Indirectly modified files (3)

These files need a one-line change to pass `customer_tags` to the `product-card` snippet:

| File | Change |
|---|---|
| `sections/product-grid.liquid` | Pass `customer_tags` to `{% render 'product-card' %}` |
| `sections/featured-products.liquid` | Pass `customer_tags` to `{% render 'product-card' %}` |
| `sections/related-products.liquid` | Pass `customer_tags` to `{% render 'product-card' %}` |

**Total: 10 new files + 12 modified files = 22 files**
