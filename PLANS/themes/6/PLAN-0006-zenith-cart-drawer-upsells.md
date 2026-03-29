# PLAN-0006: Zenith Cart Drawer with Upsells

## Overview

Add a slide-out cart drawer to the Zenith fitness theme that opens on every add-to-cart action (AJAX, no page reload), displays line items with quantity controls, an upsell product recommendations section, a free-shipping progress bar, and a checkout CTA. The drawer replaces the current "badge update + redirect" flow with an inline drawer experience while preserving the full cart page (`cart.json`) as a fallback.

---

## Pages Affected

| Page | Template | Change |
|---|---|---|
| All pages | `layout/theme.liquid` | Render the cart-drawer section globally |
| Cart page | `templates/cart.json` | No change (fallback stays intact) |
| Product page | `sections/product-detail.liquid` | Dispatch `cart:open` event after ATC |
| Collection page | `snippets/product-card.liquid` | Dispatch `cart:open` event after quick-add |
| Quick-view modal | `snippets/quick-view-modal.liquid` | Dispatch `cart:open` event after ATC |

---

## Architecture Decisions

### Why a section, not a snippet?

The cart drawer needs `{% schema %}` settings so merchants can configure upsell behavior, toggle the shipping bar, and pick a fallback upsell collection in the theme editor. Sections support this; snippets do not. The section will be rendered in `layout/theme.liquid` (outside any template) so it appears on every page.

### Why `<dialog>` element?

Aligns with the existing pattern in the theme (mobile menu, quick-view modal). Provides native focus trapping, ESC-to-close, and backdrop click handling. Accessible by default.

### Why Section Rendering API for drawer re-render?

After each cart mutation (add/change/remove), re-fetch the cart drawer section HTML via `/?sections=cart-drawer`. This gives us server-rendered Liquid (correct prices, line items, shipping bar math) without client-side templating. This is the recommended Shopify pattern for AJAX cart drawers.

### Why Product Recommendations API for upsells?

Shopify's `GET /recommendations/products.json?product_id=X&limit=4` returns AI-driven complementary products. It is free, requires no app install, and returns products that are contextually relevant to the most recently added item. Fallback: a merchant-selected collection from the section schema.

---

## Files to Create

### 1. `sections/cart-drawer.liquid` (NEW)

The main cart drawer section. Rendered globally from `layout/theme.liquid`.

**Structure:**
```
<dialog> (slide-in from right)
  <div class="cart-drawer__overlay">
  <div class="cart-drawer__panel">
    <header> Close button + "Your cart (N)" title </header>
    <div class="cart-drawer__body">
      <!-- Free shipping progress bar -->
      <!-- Line items loop -->
      <!-- Upsells section -->
    </div>
    <footer> Subtotal + Checkout CTA + Continue shopping </footer>
  </div>
</dialog>
```

**Liquid data:**
- Iterates `cart.items` for line items
- Uses `settings.free_shipping_threshold` (global setting, already exists) for the progress bar
- Renders upsell products from section setting `upsell_collection` (static fallback)
- Dynamic upsells fetched via JS Product Recommendations API on open

**JavaScript (in `<script>` tags, NOT `{% javascript %}`):**
- Listens for `cart:open` custom event to open the drawer
- Listens for `cart:updated` custom event to re-render via Section Rendering API
- Handles quantity +/- and remove via `/cart/change.js`
- After each mutation: re-fetches `/?sections=cart-drawer` and replaces inner HTML
- Fetches upsells from `/recommendations/products.json?product_id=X&limit=4&intent=complementary`
- Focus trap: on open, focus the close button; on close, return focus to trigger element
- ESC key and backdrop click close the drawer
- Dispatches `cart:updated` after every cart mutation (for header badge sync)

**CSS (in `{% stylesheet %}`):**
- Drawer slides in from right with `transform: translateX(100%)` -> `translateX(0)`
- Uses CSS variables from `critical.css` for all colors, fonts, radii, transitions
- Supports dark/light mode via existing `[data-theme]` selectors
- Backdrop uses `var(--color-overlay)` which already adapts to dark/light
- Mobile: full-width drawer; Desktop: max-width 420px
- Smooth 300ms slide animation with `cubic-bezier(0.22, 1, 0.36, 1)`

### 2. `snippets/cart-drawer-item.liquid` (NEW)

Renders a single cart line item inside the drawer.

**`{% doc %}` header:**
```liquid
{% doc %}
  Renders a single line item row inside the cart drawer. Shows product
  image thumbnail, title, variant, unit price, quantity +/- controls,
  line total, and a remove button.

  @param {line_item} item - A Shopify cart line_item object. Required.
{% enddoc %}
```

**Parameters:**
- `item` (line_item object)

**Renders:**
- 64x64 product thumbnail
- Product title (linked to product URL)
- Variant title (unless "Default Title")
- Line item properties
- Unit price + line total
- Quantity selector (reuses pattern from `cart-content.liquid`, NOT the `quantity-selector` snippet to avoid ID conflicts)
- Remove button
- `data-line-key="{{ item.key }}"` for JS targeting

**CSS:** Raw `<style>` tags (snippets cannot use `{% stylesheet %}`)

### 3. `snippets/cart-drawer-upsell.liquid` (NEW)

Renders a single upsell product card inside the drawer.

**`{% doc %}` header:**
```liquid
{% doc %}
  Renders a compact upsell product card for the cart drawer. Shows image,
  title, price, and an "Add" button that adds the first available variant
  to the cart via AJAX.

  @param {product} product - Shopify product object. Required.
{% enddoc %}
```

**Parameters:**
- `product` (product object)

**Renders:**
- 56x56 product image
- Product title (truncated to 1 line)
- Price (using `{% render 'price', product: product %}`)
- Compact "Add" button with `data-upsell-add` and `data-variant-id`

**CSS:** Raw `<style>` tags

---

## Files to Modify

### 4. `layout/theme.liquid`

**Change:** Add the cart drawer section render before `</body>`, after the footer group and before the bottom tab bar.

```liquid
{%- sections 'footer-group' -%}

{%- section 'cart-drawer' -%}

{%- render 'bottom-tab-bar' -%}
```

**Why `{%- section 'cart-drawer' -%}`:** This is a statically-rendered section. It will appear on every page and be customizable in the theme editor under the global "Cart drawer" section. Using `{% section %}` (not `{% sections %}`) makes it a single section with its own schema settings.

### 5. `sections/product-detail.liquid`

**Change:** After the ATC fetch succeeds, dispatch `cart:open` in addition to the existing `cart:updated` event.

Current (line ~1078):
```javascript
document.dispatchEvent(new CustomEvent('cart:updated', { detail: { cart: cart } }));
```

Add after:
```javascript
document.dispatchEvent(new CustomEvent('cart:open', {
  detail: { cart: cart, productId: variantId }
}));
```

Same change for the sticky ATC button (~line 1133).

### 6. `snippets/product-card.liquid`

**Change:** After the quick-add fetch succeeds (line ~416), dispatch `cart:open`.

Current:
```javascript
quickAddBtn.textContent = document.documentElement.lang === 'en'
  ? 'Added!'
  : quickAddBtn.dataset.addedLabel || 'Added!';
```

Add before the setTimeout:
```javascript
document.dispatchEvent(new CustomEvent('cart:open', {
  detail: { cart: cartData, productId: variantId }
}));
```

### 7. `snippets/quick-view-modal.liquid`

**Change:** After the ATC fetch succeeds (line ~490), dispatch `cart:open` and remove the `closeModal()` call (drawer replaces it).

Current:
```javascript
atcBtn.textContent = '{{ 'products.added_to_cart' | t | escape }}';
setTimeout(function () {
  atcBtn.textContent = originalText;
  atcBtn.disabled    = false;
  closeModal();
}, 1500);
```

Replace with:
```javascript
atcBtn.textContent = '{{ 'products.added_to_cart' | t | escape }}';
document.dispatchEvent(new CustomEvent('cart:open', {
  detail: { cart: cartData, productId: variantId }
}));
setTimeout(function () {
  atcBtn.textContent = originalText;
  atcBtn.disabled    = false;
  closeModal();
}, 800);
```

### 8. `sections/header.liquid`

**Change:** Convert the cart icon link from `<a href="{{ routes.cart_url }}">` to a `<button>` that opens the cart drawer, with the link as a fallback for no-JS.

Current (line ~167):
```html
<a href="{{ routes.cart_url }}" class="zn-header__icon-btn zn-header__cart" aria-label="{{ cart_label }}">
```

Replace with:
```html
<button
  type="button"
  class="zn-header__icon-btn zn-header__cart"
  aria-label="{{ cart_label }}"
  data-cart-drawer-toggle
>
```

Add a JS handler in the existing header `<script>`:
```javascript
var cartToggle = header.querySelector('[data-cart-drawer-toggle]');
if (cartToggle) {
  cartToggle.addEventListener('click', function () {
    document.dispatchEvent(new CustomEvent('cart:open', { detail: {} }));
  });
}
```

Also update the mobile menu footer cart link to dispatch `cart:open` instead of navigating (progressive enhancement).

### 9. `snippets/bottom-tab-bar.liquid`

**Change:** Convert the cart tab from a link to a button that dispatches `cart:open`.

Current pattern:
```html
<a href="{{ routes.cart_url }}" ...>
```

Replace with:
```html
<button type="button" data-cart-drawer-toggle ...>
```

### 10. `locales/en.default.json`

**Add keys under `cart_drawer`:**

```json
"cart_drawer": {
  "title": "Your cart",
  "title_with_count": "Your cart ({{ count }})",
  "empty": "Your cart is empty",
  "empty_cta": "Start shopping",
  "subtotal": "Subtotal",
  "shipping_note": "Shipping calculated at checkout",
  "checkout": "Checkout",
  "continue_shopping": "Continue shopping",
  "free_shipping_remaining": "Add {{ amount }} more for free shipping",
  "free_shipping_achieved": "You've unlocked free shipping!",
  "upsells_heading": "Complete your routine",
  "add": "Add",
  "adding": "Adding...",
  "remove": "Remove",
  "view_cart": "View full cart"
}
```

### 11. `locales/en.default.schema.json`

**Add under `sections`:**

```json
"cart_drawer": {
  "name": "Cart drawer",
  "settings": {
    "enable": { "label": "Enable cart drawer" },
    "show_free_shipping_bar": { "label": "Show free shipping progress bar" },
    "show_upsells": { "label": "Show upsell recommendations" },
    "upsells_heading": { "label": "Upsells heading" },
    "upsell_collection": { "label": "Fallback upsell collection" },
    "upsell_count": { "label": "Number of upsell products" }
  }
}
```

---

## Schema Settings for `sections/cart-drawer.liquid`

```json
{
  "name": "t:sections.cart_drawer.name",
  "settings": [
    {
      "type": "checkbox",
      "id": "enable",
      "label": "t:sections.cart_drawer.settings.enable.label",
      "default": true
    },
    {
      "type": "checkbox",
      "id": "show_free_shipping_bar",
      "label": "t:sections.cart_drawer.settings.show_free_shipping_bar.label",
      "default": true
    },
    {
      "type": "checkbox",
      "id": "show_upsells",
      "label": "t:sections.cart_drawer.settings.show_upsells.label",
      "default": true
    },
    {
      "type": "text",
      "id": "upsells_heading",
      "label": "t:sections.cart_drawer.settings.upsells_heading.label",
      "default": "Complete your routine"
    },
    {
      "type": "collection",
      "id": "upsell_collection",
      "label": "t:sections.cart_drawer.settings.upsell_collection.label"
    },
    {
      "type": "range",
      "id": "upsell_count",
      "label": "t:sections.cart_drawer.settings.upsell_count.label",
      "min": 2,
      "max": 6,
      "step": 1,
      "default": 4
    }
  ]
}
```

No `presets` array -- this is a statically rendered section (via `{% section %}` in layout), so it should NOT have presets. Presets are only for sections that can be added by merchants to JSON templates.

---

## JavaScript Architecture

### Event Flow

```
User clicks ATC button
  -> fetch('/cart/add.js')
  -> fetch('/cart.js') to get updated cart
  -> dispatch 'cart:updated' (badge sync in header)
  -> dispatch 'cart:open' (opens drawer)

Cart drawer opens
  -> dialog.showModal()
  -> fetch('/?sections=cart-drawer') for fresh HTML
  -> fetch('/recommendations/products.json?product_id=X') for upsells
  -> Replace drawer body innerHTML
  -> Focus close button

User changes quantity in drawer
  -> fetch('/cart/change.js')
  -> fetch('/?sections=cart-drawer') for re-render
  -> dispatch 'cart:updated' (badge sync)
  -> Replace drawer body innerHTML

User clicks upsell "Add" button
  -> fetch('/cart/add.js')
  -> fetch('/?sections=cart-drawer') for re-render
  -> dispatch 'cart:updated' (badge sync)
  -> Replace drawer body innerHTML
  -> Re-fetch recommendations (new product context)
```

### Section Rendering API

After any cart mutation, fetch the section HTML:

```javascript
fetch('/?sections=cart-drawer')
  .then(function (res) { return res.json(); })
  .then(function (data) {
    var html = data['cart-drawer'];
    var temp = document.createElement('div');
    temp.innerHTML = html;
    var newBody = temp.querySelector('[data-cart-drawer-body]');
    var currentBody = drawer.querySelector('[data-cart-drawer-body]');
    if (newBody && currentBody) {
      currentBody.innerHTML = newBody.innerHTML;
    }
  });
```

This ensures all Liquid logic (shipping bar math, price formatting, item count) is server-rendered correctly.

### Product Recommendations API

```javascript
function fetchUpsells(productId) {
  if (!productId) return;
  var url = '/recommendations/products.json?product_id=' + productId
          + '&limit=' + upsellCount + '&intent=complementary';
  fetch(url)
    .then(function (res) { return res.json(); })
    .then(function (data) {
      if (data.products && data.products.length > 0) {
        renderUpsells(data.products);
      }
    });
}
```

The `intent=complementary` parameter tells Shopify to return "frequently bought together" products rather than "related" products.

### Focus Management

```javascript
function openDrawer() {
  triggerElement = document.activeElement; // save for return focus
  dialog.showModal();
  dialog.querySelector('[data-cart-drawer-close]').focus();
  document.body.style.overflow = 'hidden';
}

function closeDrawer() {
  dialog.close();
  document.body.style.overflow = '';
  if (triggerElement) triggerElement.focus();
}
```

---

## CSS Approach

All styles in `{% stylesheet %}` inside `sections/cart-drawer.liquid` (for the section) and raw `<style>` tags inside the snippets.

### Drawer Animation

```css
.cart-drawer {
  border: none;
  padding: 0;
  margin: 0;
  position: fixed;
  inset: 0 0 0 auto;
  width: 100%;
  max-width: 420px;
  height: 100%;
  max-height: 100%;
  background: var(--color-surface);
  border-left: 1px solid var(--color-border);
  transform: translateX(100%);
  transition: transform 0.3s cubic-bezier(0.22, 1, 0.36, 1);
  z-index: var(--z-modal);
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.cart-drawer[open] {
  transform: translateX(0);
}

.cart-drawer::backdrop {
  background: var(--color-overlay);
  backdrop-filter: blur(4px);
}
```

### Dark/Light Mode

No extra work needed -- all CSS uses `var()` custom properties that already switch between dark and light in `critical.css`.

### Mobile

```css
@media (max-width: 767px) {
  .cart-drawer {
    max-width: 100%;
  }
}
```

### Progress Bar

Reuses the same visual pattern as `cart-content.liquid` shipping bar but scoped under `.cart-drawer__shipping-bar`.

---

## Integration Points

### 1. Product Page (`product-detail.liquid`)

Two ATC buttons exist: the main form button and the sticky bottom bar button. Both already dispatch `cart:updated`. We add `cart:open` dispatch to both handlers.

### 2. Collection Page (`product-card.liquid`)

The `[data-quick-add]` handler for single-variant products calls `/cart/add.js` directly. We add `cart:open` dispatch after the badge update.

For multi-variant products, the quick-add button opens the quick-view modal, which has its own ATC handler (see point 3).

### 3. Quick-View Modal (`quick-view-modal.liquid`)

The modal's ATC handler currently closes the modal after 1500ms. We shorten the delay to 800ms and dispatch `cart:open` so the drawer slides in as the modal closes.

### 4. Header Cart Icon (`header.liquid`)

Change from `<a>` to `<button>` that dispatches `cart:open`. Add `<noscript>` fallback or keep the link accessible via the "View full cart" link in the drawer footer.

### 5. Bottom Tab Bar (`bottom-tab-bar.liquid`)

Same pattern as header: convert cart tab from link to button that dispatches `cart:open`.

### 6. Cart Page (`cart-content.liquid`)

No change. The full cart page with multi-step checkout flow remains as a standalone destination. The drawer footer includes a "View full cart" link to this page.

---

## Accessibility Checklist

- [x] Uses `<dialog>` with `showModal()` for native focus trapping
- [x] ESC key closes the drawer (native dialog behavior)
- [x] Backdrop click closes the drawer
- [x] Close button is first focusable element in the drawer
- [x] Return focus to trigger element on close
- [x] `aria-label` on dialog: "Cart drawer"
- [x] `aria-live="polite"` on the cart item count and subtotal for screen reader updates
- [x] Quantity buttons have `aria-label` for increase/decrease
- [x] Remove buttons have `aria-label` with product title
- [x] Progress bar uses `role="progressbar"` with `aria-valuenow`, `aria-valuemin`, `aria-valuemax`
- [x] `body.style.overflow = 'hidden'` when open to prevent background scroll
- [x] All interactive elements have visible focus indicators (existing `:focus-visible` styles)

---

## Build Order

| # | File | Reason |
|---|---|---|
| 1 | `locales/en.default.json` | Add `cart_drawer` translation keys (needed by all subsequent files) |
| 2 | `locales/en.default.schema.json` | Add `cart_drawer` section schema labels |
| 3 | `snippets/cart-drawer-item.liquid` | Line item rendering snippet (no dependencies) |
| 4 | `snippets/cart-drawer-upsell.liquid` | Upsell card snippet (depends on `price` snippet, already exists) |
| 5 | `sections/cart-drawer.liquid` | Main drawer section (depends on #3, #4, locale keys) |
| 6 | `layout/theme.liquid` | Add `{% section 'cart-drawer' %}` render (depends on #5) |
| 7 | `sections/header.liquid` | Convert cart icon to button, add `cart:open` dispatch |
| 8 | `snippets/bottom-tab-bar.liquid` | Convert cart tab to button, add `cart:open` dispatch |
| 9 | `sections/product-detail.liquid` | Add `cart:open` dispatch after ATC |
| 10 | `snippets/product-card.liquid` | Add `cart:open` dispatch after quick-add |
| 11 | `snippets/quick-view-modal.liquid` | Add `cart:open` dispatch, shorten close delay |

---

## Translation Keys

```
cart_drawer
  title                    "Your cart"
  title_with_count         "Your cart ({{ count }})"
  empty                    "Your cart is empty"
  empty_cta                "Start shopping"
  subtotal                 "Subtotal"
  shipping_note            "Shipping calculated at checkout"
  checkout                 "Checkout"
  continue_shopping        "Continue shopping"
  free_shipping_remaining  "Add {{ amount }} more for free shipping"
  free_shipping_achieved   "You've unlocked free shipping!"
  upsells_heading          "Complete your routine"
  add                      "Add"
  adding                   "Adding..."
  remove                   "Remove"
  view_cart                "View full cart"

sections.cart_drawer       (schema labels, in en.default.schema.json)
  name                     "Cart drawer"
  settings
    enable.label           "Enable cart drawer"
    show_free_shipping_bar "Show free shipping progress bar"
    show_upsells           "Show upsell recommendations"
    upsells_heading        "Upsells heading"
    upsell_collection      "Fallback upsell collection"
    upsell_count           "Number of upsell products"
```

---

## Edge Cases to Handle

1. **Cart is empty** -- Show an empty state with an icon, message, and "Start shopping" CTA button instead of line items.

2. **Drawer disabled** -- When `section.settings.enable` is false, the section renders nothing and all `cart:open` events are ignored (ATC still works, just no drawer). Header cart icon stays as a link to `/cart`.

3. **No recommendations available** -- If the Recommendations API returns 0 products AND no fallback collection is set, hide the upsells section entirely.

4. **Product already in cart** -- Upsells should filter out products already in the cart. This filtering happens client-side after the recommendations fetch.

5. **Free shipping threshold is 0** -- If the global setting `free_shipping_threshold` is 0 or blank, hide the progress bar regardless of the section setting.

6. **Rapid quantity clicks** -- Debounce quantity change requests (300ms) to avoid race conditions with `/cart/change.js`.

7. **Sold-out variant in cart** -- If a variant becomes unavailable, the line item still shows but quantity controls are disabled. This is handled server-side by Shopify's cart object.

8. **Multiple quick-view modals and drawer** -- The drawer uses `z-index: var(--z-modal)` (400). The quick-view modal also uses z-index 400. Since the quick-view closes before/as the drawer opens (800ms delay), there is no overlap. If both are somehow open, the dialog stack order handles it natively.

---

## Estimated Effort

- **New files:** 3 (1 section, 2 snippets)
- **Modified files:** 8 (1 layout, 2 sections, 3 snippets, 2 locale files)
- **Total:** 11 files
- **Complexity:** Medium -- the JS event architecture already exists (`cart:updated`), the dialog pattern is proven in the theme, and the Section Rendering API is well-documented.
