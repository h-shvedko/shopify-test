# PLAN-0001: Predictive Search + Collection Filtering for Zenith Theme

## Overview

Add two interconnected features to the Zenith fitness theme (`themes/starter-fitness`):

1. **Predictive Search** -- An accessible overlay/dropdown that queries Shopify's Predictive Search API and shows grouped results (products, collections, articles, pages) with keyboard navigation and dark/light mode support.

2. **Collection Filtering** -- A sidebar/drawer filter system using Shopify's Storefront Filtering API with URL-based state, AJAX-powered section rendering, active filter pills, sort-by dropdown, and responsive layout (sidebar on desktop, drawer on mobile).

---

## Pages Affected

| Page | Template | Sections Affected |
|---|---|---|
| All pages | `layout/theme.liquid` | Header (search trigger) |
| Collection | `templates/collection.json` | `product-grid` (filters, sort, AJAX) |
| Search | `templates/search.json` | `search-results` (minor: link to predictive) |

---

## Part 1: Predictive Search

### Architecture

The predictive search is a **snippet** (no schema needed) rendered inside the header section. It consists of:
- A search trigger button (already exists in header -- the search icon link)
- A `<dialog>` overlay containing the search input and results container
- JavaScript that calls `/search/suggest.json` with debounced input

### Why a snippet, not a section

Predictive search is embedded within the header section. It does not need its own theme editor settings or appear as a standalone sortable block. The header section already has the search icon; we convert it from a link to a button that opens the search dialog. The snippet receives no Liquid data at render time -- everything is fetched via JavaScript.

### Files to Create

#### 1. `snippets/predictive-search.liquid`

A self-contained search overlay component.

**HTML structure:**
```
<dialog class="zn-search" data-predictive-search>
  <div class="zn-search__inner">
    <form action="/search" method="get" role="search">
      <div class="zn-search__input-wrap" role="combobox" aria-expanded="false" aria-haspopup="listbox" aria-owns="predictive-search-results">
        <svg search icon />
        <input type="search" name="q" autocomplete="off" aria-autocomplete="list" aria-controls="predictive-search-results" />
        <button type="button" close icon, data-search-close />
      </div>
    </form>
    <div id="predictive-search-results" class="zn-search__results" role="listbox" aria-label="{{ t key }}">
      <!-- JS-populated results grouped by type -->
    </div>
    <div class="zn-search__status" aria-live="polite" role="status"></div>
  </div>
</dialog>
```

**`{% doc %}` header:**
```liquid
{% doc %}
  Renders the predictive search overlay dialog.
  Uses Shopify's /search/suggest.json API via JavaScript.
  No parameters -- call as {% render 'predictive-search' %}.

  Accessibility:
  - Combobox pattern (aria-expanded, aria-autocomplete, aria-controls)
  - aria-live region for result count announcements
  - Keyboard navigation (arrow keys, Enter, Escape)

  Usage:
    {% render 'predictive-search' %}
{% enddoc %}
```

**CSS (raw `<style>` tags per snippet convention):**
- `.zn-search` -- full-screen dialog, z-index: 400 (matches `--z-modal`)
- `.zn-search__inner` -- max-width 720px, centered, glassmorphism background
- `.zn-search__input-wrap` -- input with search icon left, close button right
- `.zn-search__results` -- scrollable results area, max-height 60vh
- `.zn-search__group` -- each result type group (products, collections, etc.)
- `.zn-search__group-title` -- eyebrow-style category label
- `.zn-search__item` -- individual result row, flex layout
- `.zn-search__item--active` -- keyboard-focused highlight (teal border/bg)
- `.zn-search__product-thumb` -- 48x48 thumbnail
- `.zn-search__product-info` -- title + price
- `.zn-search__empty` -- "No results" state
- `.zn-search__loading` -- spinner during fetch
- All colors use CSS custom properties (works in dark AND light mode automatically)

**JavaScript (`<script>` tag per convention):**

```
Core behavior:
1. Open dialog on trigger click (data-search-open)
2. On input, debounce 300ms, then fetch:
   GET /search/suggest.json?q={query}&resources[type]=product,collection,article,page&resources[limit]=4
3. Parse response, group by type, render HTML into results container
4. Product results: thumbnail (48px), title (line-clamped), price (money formatted)
5. Collection/article/page results: title + type badge
6. Keyboard navigation:
   - ArrowDown/ArrowUp: move through results (data-search-index)
   - Enter: navigate to focused result URL
   - Escape: close dialog
   - Tab: trapped inside dialog while open
7. Click result: navigate to URL
8. Close: dialog.close(), clear results, refocus trigger
9. aria-expanded toggles on combobox wrapper
10. aria-live region announces "N results" or "No results"
```

**Money formatting:** Use `Shopify.formatMoney()` if available, otherwise a simple formatter that reads `window.Shopify.locale` and `window.Shopify.currency.active`.

#### 2. Modify `sections/header.liquid`

**Changes:**
1. Convert the search `<a>` link (lines 155-164) from an anchor to a `<button>`:
   ```liquid
   <button
     type="button"
     class="zn-header__icon-btn"
     aria-label="{{ search_label }}"
     data-search-open
   >
     <!-- same SVG -->
   </button>
   ```

2. Add the predictive search render call after the header closing tag but before the mobile menu dialog:
   ```liquid
   {%- render 'predictive-search' -%}
   ```

3. Add a new schema setting to enable/disable predictive search:
   ```json
   {
     "type": "checkbox",
     "id": "enable_predictive_search",
     "label": "t:sections.header.settings.enable_predictive_search.label",
     "default": true
   }
   ```

4. Conditionally render:
   ```liquid
   {%- if section.settings.enable_predictive_search -%}
     {%- render 'predictive-search' -%}
   {%- endif -%}
   ```
   When disabled, the search button becomes a link to `/search` instead.

### Predictive Search Schema Settings

No dedicated section schema (it is a snippet). The on/off toggle lives in the header section schema. Result count limits and resource types are hardcoded (4 per type) since merchants rarely need to configure these.

---

## Part 2: Collection Filtering

### Architecture

Collection filtering is implemented as a major enhancement to the existing `sections/product-grid.liquid` section. It uses:

- Shopify's `collection.filters` object for available filter values
- URL query parameters for filter state (`filter.v.availability`, `filter.v.price`, `filter.p.vendor`, etc.)
- The Section Rendering API for AJAX-powered filtering without full page reload
- A `<dialog>` drawer for mobile filters, a sidebar layout for desktop

### Files to Create/Modify

#### 1. Create `snippets/collection-filters.liquid`

Renders the filter form with all active filters from the collection object.

**`{% doc %}` header:**
```liquid
{% doc %}
  Renders collection filter controls using Shopify's Storefront Filtering API.

  @param {collection} collection - The current collection object. Required.
  @param {string} section_id - Parent section ID for AJAX targeting. Required.
  @param {boolean} [show_price_range] - Show price range filter. Default: true.
  @param {boolean} [show_availability] - Show availability filter. Default: true.

  Usage:
    {%- assign coll = collection -%}
    {% render 'collection-filters', collection: coll, section_id: section.id %}
{% enddoc %}
```

**HTML structure:**
```html
<form data-collection-filters data-section-id="{{ section_id }}">
  <!-- Active filter pills -->
  <div class="zn-filters__active" data-active-filters>
    {% for filter in collection.filters %}
      {% for value in filter.active_values %}
        <a href="{{ value.url_to_remove }}" class="zn-filters__pill" data-filter-remove>
          {{ value.label }} <svg x-icon />
        </a>
      {% endfor %}
    {% endfor %}
    {% if has_active_filters %}
      <a href="{{ collection.url }}?sort_by={{ collection.sort_by }}" class="zn-filters__clear-all">
        {{ 'collection.clear_filters' | t }}
      </a>
    {% endif %}
  </div>

  <!-- Filter groups -->
  {% for filter in collection.filters %}
    <details class="zn-filters__group" {% if filter.active_values.size > 0 %}open{% endif %}>
      <summary class="zn-filters__group-title">
        {{ filter.label }}
        {% if filter.active_values.size > 0 %}
          <span class="zn-filters__group-count">{{ filter.active_values.size }}</span>
        {% endif %}
      </summary>
      <div class="zn-filters__group-body">
        {% case filter.type %}
          {% when 'list' %}
            <!-- Checkbox list -->
            {% for value in filter.values %}
              <label class="zn-filters__option">
                <input
                  type="checkbox"
                  name="{{ value.param_name }}"
                  value="{{ value.value }}"
                  {% if value.active %}checked{% endif %}
                  {% if value.count == 0 %}disabled{% endif %}
                  data-filter-input
                >
                <span class="zn-filters__option-label">{{ value.label }}</span>
                <span class="zn-filters__option-count">({{ value.count }})</span>
              </label>
            {% endfor %}

          {% when 'price_range' %}
            <!-- Price range inputs -->
            <div class="zn-filters__price-range">
              <div class="zn-filters__price-field">
                <label for="Filter-{{ filter.label | handleize }}-min">{{ 'collection.filter_price_from' | t }}</label>
                <input
                  type="number"
                  id="Filter-{{ filter.label | handleize }}-min"
                  name="{{ filter.min_value.param_name }}"
                  value="{{ filter.min_value.value | money_without_currency | replace: ',', '' }}"
                  min="0"
                  max="{{ filter.range_max | money_without_currency | replace: ',', '' }}"
                  placeholder="0"
                  data-filter-input
                  data-filter-price
                >
              </div>
              <span class="zn-filters__price-separator">--</span>
              <div class="zn-filters__price-field">
                <label for="Filter-{{ filter.label | handleize }}-max">{{ 'collection.filter_price_to' | t }}</label>
                <input
                  type="number"
                  id="Filter-{{ filter.label | handleize }}-max"
                  name="{{ filter.max_value.param_name }}"
                  value="{{ filter.max_value.value | money_without_currency | replace: ',', '' }}"
                  min="0"
                  max="{{ filter.range_max | money_without_currency | replace: ',', '' }}"
                  placeholder="{{ filter.range_max | money_without_currency }}"
                  data-filter-input
                  data-filter-price
                >
              </div>
            </div>
        {% endcase %}
      </div>
    </details>
  {% endfor %}
</form>
```

**CSS (raw `<style>` tags):**
- `.zn-filters__active` -- flex-wrap pill container
- `.zn-filters__pill` -- badge-style pill with X icon, teal border, remove on click
- `.zn-filters__clear-all` -- text link, coral color
- `.zn-filters__group` -- `<details>` with animated open/close
- `.zn-filters__group-title` -- `<summary>` styled as bold label with chevron
- `.zn-filters__group-count` -- teal badge showing active count
- `.zn-filters__option` -- checkbox row with label and count
- `.zn-filters__option input[type="checkbox"]` -- custom checkbox matching Zenith's teal accent
- `.zn-filters__option--disabled` -- dimmed for zero-count options
- `.zn-filters__price-range` -- flex row with min/max inputs
- All colors use CSS custom properties for dark/light mode

#### 2. Create `snippets/collection-sort.liquid`

Renders the sort-by dropdown. Extracted from product-grid to be reusable.

**`{% doc %}` header:**
```liquid
{% doc %}
  Renders a sort-by dropdown for collection pages.

  @param {string} current_sort - Current sort_by value. Required.
  @param {string} section_id - Parent section ID. Required.

  Usage:
    {% render 'collection-sort', current_sort: collection.sort_by, section_id: section.id %}
{% enddoc %}
```

#### 3. Major rewrite of `sections/product-grid.liquid`

This is the most substantial change. The section gains a two-column layout on desktop (sidebar filters + product grid) and AJAX filtering capability.

**Layout restructure:**

```html
<section class="product-grid-section" data-section-id="{{ section.id }}" id="ProductGrid-{{ section.id }}">
  <div class="product-grid__container">

    <!-- Mobile filter trigger -->
    <div class="product-grid__mobile-controls">
      <button type="button" class="btn btn--ghost btn--sm" data-filter-drawer-open>
        <svg filter-icon /> {{ 'collection.filter' | t }}
        {% if active_filter_count > 0 %}
          <span class="zn-filters__count-badge">{{ active_filter_count }}</span>
        {% endif %}
      </button>
      {% render 'collection-sort', current_sort: collection.sort_by, section_id: section.id %}
    </div>

    <div class="product-grid__layout">
      <!-- Desktop sidebar -->
      <aside class="product-grid__sidebar" data-filter-sidebar>
        {% render 'collection-filters', collection: collection, section_id: section.id %}
      </aside>

      <!-- Main content -->
      <div class="product-grid__main">
        <!-- Toolbar: count + sort (desktop) -->
        <div class="product-grid__toolbar">
          <p class="product-grid__count" data-product-count>
            {{ 'products.showing_count' | t: count: collection.products_count }}
          </p>
          <div class="product-grid__toolbar-right">
            {% render 'collection-sort', current_sort: collection.sort_by, section_id: section.id %}
          </div>
        </div>

        <!-- Active filter pills (desktop, above grid) -->
        <div class="product-grid__active-filters" data-active-filters-desktop>
          <!-- Rendered from collection-filters snippet or duplicated here -->
        </div>

        <!-- Product grid -->
        <div class="product-grid__grid ..." data-products-container>
          {% for product in collection.products %}
            {% render 'product-card', ... %}
          {% endfor %}
        </div>

        <!-- Pagination -->
        ...
      </div>
    </div>
  </div>
</section>

<!-- Mobile filter drawer -->
<dialog class="zn-filter-drawer" data-filter-drawer>
  <div class="zn-filter-drawer__inner">
    <div class="zn-filter-drawer__header">
      <h2>{{ 'collection.filter' | t }}</h2>
      <button data-filter-drawer-close>X</button>
    </div>
    <div class="zn-filter-drawer__body">
      {% render 'collection-filters', collection: collection, section_id: section.id %}
    </div>
    <div class="zn-filter-drawer__footer">
      <button class="btn btn--primary btn--full-width" data-filter-apply>
        {{ 'collection.show_results' | t: count: collection.products_count }}
      </button>
    </div>
  </div>
</dialog>
```

**New CSS (within `{% stylesheet %}`):**
- `.product-grid__layout` -- CSS grid, 1 column on mobile, `260px 1fr` on desktop (min-width: 1024px)
- `.product-grid__sidebar` -- hidden on mobile, shown on desktop when `show_filter` enabled
- `.product-grid__mobile-controls` -- flex row, shown on mobile only, hidden on desktop
- `.zn-filter-drawer` -- full-height dialog from left (same pattern as mobile menu)
- `.zn-filter-drawer__inner` -- flex-column, header/body(scrollable)/footer
- `.zn-filter-drawer__footer` -- sticky bottom with "Show N results" button
- Active filter pills row above grid

**New schema settings:**
```json
{
  "type": "checkbox",
  "id": "show_filter",
  "label": "t:sections.product_grid.settings.show_filters.label",
  "default": true
},
{
  "type": "checkbox",
  "id": "show_product_count",
  "label": "t:sections.product_grid.settings.show_product_count.label",
  "default": true
},
{
  "type": "select",
  "id": "filter_layout",
  "label": "t:sections.product_grid.settings.filter_layout.label",
  "options": [
    { "value": "sidebar", "label": "t:sections.product_grid.settings.filter_layout.sidebar" },
    { "value": "horizontal", "label": "t:sections.product_grid.settings.filter_layout.horizontal" }
  ],
  "default": "sidebar"
}
```

**JavaScript (`<script>` tag) -- AJAX filtering engine:**

```
Core behavior:
1. Listen for change events on [data-filter-input] elements
2. On change (checkbox/price input), build URL from form serialization:
   - Collect all checked checkboxes and non-empty price inputs
   - Preserve sort_by param
   - Build query string: ?filter.v.availability=1&filter.v.price.gte=20&...&sort_by=...
3. Push new URL to browser history (history.pushState)
4. Fetch the new URL with Section Rendering API:
   GET {url}?sections={section_id}
   Response returns HTML for just this section
5. Extract the product grid, toolbar, pagination, active filters, and filter form
   from the returned HTML and swap them into the DOM
6. Re-initialize product card JS (swipe, quick-add, compare)
7. Re-initialize the scroll reveal observer for new cards
8. Update the product count display
9. Handle browser back/forward with popstate listener

Sort-by integration:
- Sort dropdown change builds URL with sort_by param + existing filter params
- Same AJAX fetch + DOM swap flow

Mobile filter drawer:
- Open drawer button dispatches dialog.showModal()
- "Show results" button closes drawer + triggers AJAX fetch
- On mobile, filter changes do NOT auto-fetch; they accumulate until "Show results"
- On desktop, filter changes auto-fetch with 500ms debounce for price range inputs

Price range:
- Debounce 800ms after last keystroke before triggering fetch
- Validate min <= max

Filter pill removal:
- Click pill -> navigate to value.url_to_remove via AJAX fetch
- "Clear all" -> navigate to collection URL (preserving sort) via AJAX

Loading state:
- Add .is-loading class to product grid container during fetch
- Show spinner overlay
- Reduce opacity of grid to 0.5

Error handling:
- On fetch failure, fallback to full page navigation (window.location.href = url)
```

#### 4. Modify `templates/collection.json`

Update settings to enable new features:
```json
{
  "sections": {
    "main": {
      "type": "product-grid",
      "settings": {
        "products_per_page": 12,
        "columns_desktop": "3",
        "show_sort": true,
        "show_filter": true,
        "show_product_count": true,
        "filter_layout": "sidebar"
      }
    }
  },
  "order": ["main"]
}
```

---

## Part 3: Translation Keys

### New keys for `locales/en.default.json`

```json
{
  "predictive_search": {
    "title": "Search",
    "placeholder": "Search programs, gear & more...",
    "close": "Close search",
    "products": "Products",
    "collections": "Collections",
    "articles": "Articles",
    "pages": "Pages",
    "no_results": "No results for \"{{ query }}\"",
    "results_count": "{{ count }} results found",
    "view_all": "View all results for \"{{ query }}\"",
    "loading": "Searching..."
  },
  "collection": {
    "filter": "Filter",
    "sort": "Sort",
    "sort_options": "Sort options",
    "products_count": "{{ count }} products",
    "empty": "No products match your filters",
    "clear_filters": "Clear all filters",
    "clear_filter": "Remove filter: {{ label }}",
    "filter_price_from": "From",
    "filter_price_to": "To",
    "show_results": "Show {{ count }} results",
    "active_filters": "Active filters",
    "filter_by": "Filter by {{ label }}",
    "loading_results": "Loading results...",
    "compare_selected": "{{ count }} items selected",
    "clear_compare": "Clear compare"
  }
}
```

Note: The existing `collection` key already has some entries (`filter`, `sort`, `products_count`, `empty`, `compare_selected`, `clear_compare`). The new keys merge into the existing object -- do NOT overwrite existing keys, only add new ones.

### New keys for `locales/en.default.schema.json`

```json
{
  "sections": {
    "header": {
      "settings": {
        "enable_predictive_search": {
          "label": "Enable predictive search"
        }
      }
    },
    "product_grid": {
      "settings": {
        "show_product_count": {
          "label": "Show product count"
        },
        "filter_layout": {
          "label": "Filter layout",
          "sidebar": "Sidebar",
          "horizontal": "Horizontal bar"
        }
      }
    }
  }
}
```

---

## Part 4: CSS to Add to `assets/critical.css`

Add a small set of shared filter/search utility styles at the end of critical.css. These are shared between the predictive search snippet and the collection filters snippet.

```css
/* ============================================================
   31. Filter & Search Shared
   ============================================================ */

/* Loading overlay for AJAX content swap */
.is-loading {
  position: relative;
  opacity: 0.5;
  pointer-events: none;
  transition: opacity var(--transition-base);
}

.is-loading::after {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 24px;
  height: 24px;
  margin: -12px 0 0 -12px;
  border: 2px solid var(--color-border);
  border-top-color: var(--color-accent);
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
  z-index: 10;
}

/* Custom checkbox (filters) */
.zn-checkbox {
  appearance: none;
  width: 18px;
  height: 18px;
  border: 1.5px solid var(--color-border);
  border-radius: 4px;
  background: var(--color-surface);
  cursor: pointer;
  flex-shrink: 0;
  transition:
    background var(--transition-fast),
    border-color var(--transition-fast);
}

.zn-checkbox:checked {
  background: var(--color-accent);
  border-color: var(--color-accent);
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 24 24' fill='none' stroke='%230A0A0A' stroke-width='3' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpolyline points='20 6 9 17 4 12'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: center;
}

.zn-checkbox:focus-visible {
  outline: 2px solid var(--color-accent);
  outline-offset: 2px;
}

.zn-checkbox:disabled {
  opacity: 0.35;
  cursor: not-allowed;
}
```

---

## Part 5: Complete File List

### Files to Create (4)

| # | File | Type | Purpose |
|---|---|---|---|
| 1 | `snippets/predictive-search.liquid` | Snippet | Predictive search overlay dialog with JS |
| 2 | `snippets/collection-filters.liquid` | Snippet | Filter form rendering (checkbox lists, price range) |
| 3 | `snippets/collection-sort.liquid` | Snippet | Sort-by dropdown (extracted from product-grid) |
| 4 | `snippets/filter-pill.liquid` | Snippet | Single active filter pill (reusable) |

### Files to Modify (5)

| # | File | Change |
|---|---|---|
| 1 | `sections/header.liquid` | Convert search link to button, add predictive search render, add schema setting |
| 2 | `sections/product-grid.liquid` | Add sidebar/drawer layout, filter rendering, AJAX filtering JS, new schema settings |
| 3 | `templates/collection.json` | Add new default settings (show_filter, filter_layout) |
| 4 | `locales/en.default.json` | Add predictive_search keys, extend collection keys |
| 5 | `locales/en.default.schema.json` | Add schema labels for new settings |
| 6 | `assets/critical.css` | Add .is-loading overlay and .zn-checkbox shared styles |

---

## Part 6: Build Order

The build order is based on dependencies -- foundational files first, then files that depend on them.

| Step | File | Reason |
|---|---|---|
| 1 | `locales/en.default.json` | All Liquid files reference translation keys -- must exist first |
| 2 | `locales/en.default.schema.json` | Schema labels for new settings |
| 3 | `assets/critical.css` | Shared .is-loading and .zn-checkbox styles used by snippets |
| 4 | `snippets/filter-pill.liquid` | Smallest reusable unit, used by collection-filters |
| 5 | `snippets/collection-sort.liquid` | Sort dropdown, used by product-grid |
| 6 | `snippets/collection-filters.liquid` | Filter form, depends on filter-pill, used by product-grid |
| 7 | `snippets/predictive-search.liquid` | Self-contained, no Liquid dependencies |
| 8 | `sections/product-grid.liquid` | Depends on collection-filters, collection-sort snippets |
| 9 | `sections/header.liquid` | Depends on predictive-search snippet |
| 10 | `templates/collection.json` | Update default settings for product-grid section |

---

## Part 7: JavaScript Architecture

### Predictive Search JS (inside `snippets/predictive-search.liquid`)

```
Approach: Single IIFE, no classes, vanilla JS

State:
- currentQuery (string)
- debounceTimer (timeout id)
- activeIndex (number, -1 = none)
- results (array of { url, title, type, ... })
- isOpen (boolean, tracked via dialog.open)

API call:
  fetch(`/search/suggest.json?q=${encodeURIComponent(query)}&resources[type]=product,collection,article,page&resources[limit]=4&resources[options][unavailable_products]=last`)
  .then(r => r.json())
  .then(data => {
    // data.resources.results has: products, collections, articles, pages
  })

DOM update:
  Build HTML string for each group, set innerHTML on results container.
  Each result item gets data-search-index="N" and role="option".

Keyboard handling:
  input keydown:
    ArrowDown -> activeIndex++, scroll into view, set aria-activedescendant
    ArrowUp -> activeIndex--, scroll into view
    Enter -> if activeIndex >= 0, navigate to results[activeIndex].url
    Escape -> close dialog

Focus trap:
  On dialog open, focus input.
  On dialog close, restore focus to trigger button.
```

### Collection Filtering JS (inside `sections/product-grid.liquid`)

```
Approach: Single IIFE, no classes, vanilla JS

URL building:
  function buildFilterUrl() {
    var params = new URLSearchParams();
    // Collect all checked checkboxes
    form.querySelectorAll('[data-filter-input]:checked').forEach(cb => {
      params.append(cb.name, cb.value);
    });
    // Collect price range values
    form.querySelectorAll('[data-filter-price]').forEach(input => {
      if (input.value) params.set(input.name, input.value * 100); // Shopify expects cents
    });
    // Preserve sort
    if (currentSort) params.set('sort_by', currentSort);
    return window.location.pathname + '?' + params.toString();
  }

Section Rendering API:
  function fetchAndRender(url) {
    container.classList.add('is-loading');
    fetch(url + (url.includes('?') ? '&' : '?') + 'sections=' + sectionId)
    .then(r => r.json())
    .then(data => {
      var html = data[sectionId];
      var doc = new DOMParser().parseFromString(html, 'text/html');
      // Swap: grid, toolbar, pagination, active filters, filter form
      swapElement('[data-products-container]', doc);
      swapElement('[data-product-count]', doc);
      swapElement('[data-active-filters]', doc);
      // Re-init product cards
      doc.querySelectorAll('.product-card').forEach(initProductCard);
      // Update filter sidebar (counts change)
      swapElement('[data-filter-sidebar]', doc);
      container.classList.remove('is-loading');
    })
    .catch(() => {
      window.location.href = url; // fallback
    });
    history.pushState(null, '', url);
  }

popstate handler:
  window.addEventListener('popstate', () => {
    fetchAndRender(window.location.href);
  });

Desktop vs Mobile behavior:
  var isMobile = window.matchMedia('(max-width: 1023px)');
  - Desktop: auto-fetch on each checkbox change, debounced 800ms for price
  - Mobile: accumulate changes, fetch only on "Show results" button click
```

---

## Part 8: Integration Points

### Header Integration
- The search icon button in the header triggers `predictive-search` dialog
- When `enable_predictive_search` is false, it reverts to a plain link to `/search`
- The predictive search "View all results" link navigates to the full search results page

### Collection Template Integration
- The `product-grid` section reads `collection.filters` (Shopify's Storefront Filtering API)
- Filters are only available on collection pages (not on search results)
- The sort dropdown works on both collection and search pages
- Filter state is entirely URL-based -- bookmarkable, shareable, SEO-friendly

### Dark/Light Mode Integration
- All new CSS uses existing CSS custom properties (`--color-base`, `--color-surface`, `--color-accent`, etc.)
- The `[data-theme="light"]` overrides in `critical.css` automatically apply
- No additional dark/light mode CSS is needed beyond using the variables

### Bottom Tab Bar Integration
- The bottom tab bar's "Search" icon (`snippets/bottom-tab-bar.liquid`) should also trigger the predictive search dialog on mobile, not navigate to `/search`
- Add `data-search-open` attribute to the search tab bar button

### Product Card Re-initialization
- After AJAX filtering swaps DOM content, product card JS (swipe, quick-add, compare) must be re-initialized
- The existing `initProductCard()` function in `snippets/product-card.liquid` is already scoped per card -- call it on new cards after swap
- The MutationObserver in `layout/theme.liquid` (scroll reveal) already handles dynamically added `.reveal` elements

### Search Results Page
- The existing `sections/search-results.liquid` remains unchanged for full search
- The predictive search overlay links to the full search page via the "View all results" link
- Future enhancement: add filtering to search results page (out of scope for this plan)

---

## Part 9: Accessibility Checklist

### Predictive Search
- [x] `role="combobox"` on input wrapper
- [x] `aria-expanded` toggles with results visibility
- [x] `aria-autocomplete="list"` on input
- [x] `aria-controls` points to results container
- [x] `aria-activedescendant` updates with keyboard navigation
- [x] `role="listbox"` on results container
- [x] `role="option"` on each result item
- [x] `aria-live="polite"` status region announces result count
- [x] Escape key closes dialog
- [x] Focus trapped in dialog while open
- [x] Focus returns to trigger button on close

### Collection Filters
- [x] `<details>`/`<summary>` for collapsible filter groups (native keyboard support)
- [x] Proper `<label>` association for all checkboxes and inputs
- [x] `aria-label` on filter drawer dialog
- [x] `aria-live="polite"` on product count (updates after AJAX)
- [x] Disabled checkboxes for zero-count filter values (grayed out, not removed)
- [x] Filter pill removal links have descriptive `aria-label`
- [x] Loading state communicated via `aria-busy="true"` on grid container

---

## Part 10: Performance Considerations

1. **Debouncing** -- 300ms for predictive search input, 800ms for price range inputs
2. **Section Rendering API** -- Only fetches HTML for the one section, not the full page
3. **No external libraries** -- All vanilla JS, no added bundle weight
4. **Lazy image loading** -- Product card images keep `loading="lazy"`
5. **Minimal DOM manipulation** -- Use `innerHTML` swap (not individual element creation) for filter results
6. **Cache-friendly URLs** -- Filter state in URL params enables browser caching
7. **Progressive enhancement** -- If JS fails, checkboxes submit the form normally (full page reload), sort dropdown works via page navigation, search link goes to `/search`

---

## Estimated File Sizes

| File | Estimated Lines |
|---|---|
| `snippets/predictive-search.liquid` | ~350 (HTML + style + script) |
| `snippets/collection-filters.liquid` | ~200 (HTML + style) |
| `snippets/collection-sort.liquid` | ~60 |
| `snippets/filter-pill.liquid` | ~30 |
| `sections/product-grid.liquid` (rewrite) | ~900 (up from ~736, adding layout + AJAX JS) |
| `sections/header.liquid` (modifications) | ~15 lines changed |
| `assets/critical.css` (additions) | ~50 lines added |
| `locales/en.default.json` (additions) | ~25 keys added |
| `locales/en.default.schema.json` (additions) | ~10 keys added |
