# PLAN-0001: Zenith Theme SEO Optimization Showcase

## Overview

Add comprehensive SEO infrastructure to the Zenith (starter-fitness) theme. The theme currently has zero structured data, no Open Graph / Twitter Card meta tags, and no preconnect hints. The only existing SEO elements are a basic `<title>` tag, a `<meta name="description">`, and a `<link rel="canonical">` in `layout/theme.liquid`.

This plan adds JSON-LD structured data for every page type, full social sharing meta tags, performance resource hints, and merchant-configurable SEO settings -- all following Shopify OS 2.0 conventions.

---

## Current State Audit

### What exists
| Element | Location | Status |
|---|---|---|
| `<title>` with pagination/tags | `layout/theme.liquid:9-13` | Good |
| `<meta name="description">` | `layout/theme.liquid:15-17` | Good |
| `<link rel="canonical">` | `layout/theme.liquid:19` | Good |
| Breadcrumb nav (visual only) | `sections/product-detail.liquid:759-767` | No structured data |
| FAQ with `<details>/<summary>` | `sections/faq-section.liquid` | No FAQPage schema |
| Image `loading="lazy"` | Most sections | Good coverage |
| Hero image `fetchpriority="high"` | `sections/hero-banner.liquid:32` | Good |
| Product ratings metafield | `sections/product-detail.liquid:778-779` | Used for display, not in schema |

### What is missing
- All JSON-LD structured data (Product, Organization, BreadcrumbList, FAQPage, CollectionPage, WebSite + SearchAction)
- Open Graph meta tags (og:title, og:description, og:image, og:type, og:url, og:site_name)
- Twitter Card meta tags (twitter:card, twitter:title, twitter:description, twitter:image)
- Robots meta for paginated/filtered pages
- Preconnect hints for Shopify CDN
- Global SEO settings in theme config (Organization name, logo, social profiles)
- No blog/article templates exist (skip Article/BlogPosting schema)

---

## Files to Create

### 1. `snippets/seo-meta-tags.liquid` (NEW)

Enhanced meta tags snippet rendered in `<head>`. Handles Open Graph, Twitter Cards, robots meta, and canonical URL edge cases.

**What it outputs:**
```html
<!-- Open Graph -->
<meta property="og:site_name" content="{{ shop.name }}">
<meta property="og:url" content="{{ canonical_url }}">
<meta property="og:title" content="{{ page_title }}">
<meta property="og:description" content="{{ page_description | default: shop.description }}">
<meta property="og:type" content="..."> <!-- website|product|article depending on template -->
<meta property="og:image" content="..."> <!-- product image, collection image, or shop logo -->
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta property="og:locale" content="{{ request.locale.iso_code | replace: '-', '_' }}">

<!-- Product-specific OG -->
<meta property="og:price:amount" content="...">
<meta property="og:price:currency" content="...">
<meta property="product:availability" content="...">

<!-- Twitter Card -->
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="{{ page_title }}">
<meta name="twitter:description" content="{{ page_description | default: shop.description }}">
<meta name="twitter:image" content="...">

<!-- Robots -->
<meta name="robots" content="noindex, follow"> <!-- only for paginated/filtered pages -->
```

**Logic:**
- `og:type` = `product` on product templates, `website` everywhere else
- `og:image` priority: product featured image > collection image > `settings.seo_default_image` > nothing
- Robots `noindex` when `current_page > 1` OR when `current_tags` is not empty (filtered collection pages)
- Twitter card handle from `settings.seo_twitter_handle` if set

**`{% doc %}` header params:**
- None (uses global Shopify objects only)

---

### 2. `snippets/seo-jsonld-organization.liquid` (NEW)

Sitewide Organization + WebSite schema. Rendered once in layout.

**JSON-LD output:**
```json
[
  {
    "@context": "https://schema.org",
    "@type": "Organization",
    "name": "{{ shop.name }}",
    "url": "{{ shop.url }}",
    "logo": "...",
    "sameAs": ["instagram", "facebook", "twitter", "youtube", "tiktok"],
    "contactPoint": {
      "@type": "ContactPoint",
      "email": "{{ shop.email }}",
      "contactType": "customer service"
    }
  },
  {
    "@context": "https://schema.org",
    "@type": "WebSite",
    "name": "{{ shop.name }}",
    "url": "{{ shop.url }}",
    "potentialAction": {
      "@type": "SearchAction",
      "target": "{{ shop.url }}/search?q={search_term_string}",
      "query-input": "required name=search_term_string"
    }
  }
]
```

**Data sources:**
- `shop.name`, `shop.url`, `shop.email`
- Social URLs from existing `settings.social_instagram`, `settings.social_facebook`, `settings.social_twitter`, `settings.social_youtube`, `settings.social_tiktok`
- Logo from new `settings.seo_organization_logo` (falls back to header logo)

---

### 3. `snippets/seo-jsonld-product.liquid` (NEW)

Product schema for product pages.

**JSON-LD output:**
```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "{{ product.title }}",
  "description": "{{ product.description | strip_html | truncate: 5000 }}",
  "image": ["array of product image URLs"],
  "sku": "{{ variant.sku }}",
  "brand": {
    "@type": "Brand",
    "name": "{{ product.vendor }}"
  },
  "offers": {
    "@type": "AggregateOffer",
    "priceCurrency": "{{ cart.currency.iso_code }}",
    "lowPrice": "{{ product.price_min | money_without_currency }}",
    "highPrice": "{{ product.price_max | money_without_currency }}",
    "offerCount": "{{ product.variants.size }}",
    "availability": "https://schema.org/InStock",
    "url": "{{ shop.url }}{{ product.url }}"
  },
  "aggregateRating": { ... } // only if reviews metafield exists
}
```

**Conditional logic:**
- If only one variant, use `Offer` instead of `AggregateOffer`
- Include `aggregateRating` only when `product.metafields.reviews.rating` is present
- Map `product.available` to `InStock` / `OutOfStock`
- Include up to 10 product images

**`{% doc %}` header params:**
- `product` (product object) -- required

---

### 4. `snippets/seo-jsonld-breadcrumb.liquid` (NEW)

BreadcrumbList schema. Used on product and collection pages.

**JSON-LD output:**
```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "Home", "item": "{{ shop.url }}" },
    { "@type": "ListItem", "position": 2, "name": "{{ collection.title }}", "item": "{{ shop.url }}{{ collection.url }}" },
    { "@type": "ListItem", "position": 3, "name": "{{ product.title }}" }
  ]
}
```

**Logic:**
- Always starts with "Home"
- On collection pages: Home > Collection title
- On product pages: Home > Collection (if present via `collection` object) > Product title
- Last item omits `item` URL per Google spec (current page)

**`{% doc %}` header params:**
- None (uses global `collection`, `product` objects)

---

### 5. `snippets/seo-jsonld-collection.liquid` (NEW)

CollectionPage + ItemList schema for collection pages.

**JSON-LD output:**
```json
{
  "@context": "https://schema.org",
  "@type": "CollectionPage",
  "name": "{{ collection.title }}",
  "description": "{{ collection.description | strip_html }}",
  "url": "{{ shop.url }}{{ collection.url }}",
  "mainEntity": {
    "@type": "ItemList",
    "numberOfItems": "{{ collection.products_count }}",
    "itemListElement": [
      // first 12 products as ListItem with position, url, name, image
    ]
  }
}
```

**Logic:**
- Limit ItemList to first 12 products (current page) to keep payload small
- Only include `image` if product has a featured image

**`{% doc %}` header params:**
- `collection` (collection object) -- required

---

### 6. `snippets/seo-jsonld-faq.liquid` (NEW)

FAQPage schema generated from FAQ section blocks.

**JSON-LD output:**
```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "{{ block.settings.question }}",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "{{ block.settings.answer | strip_html }}"
      }
    }
  ]
}
```

**Integration point:** Rendered inside `sections/faq-section.liquid`, after the closing `</section>` tag and before `{% stylesheet %}`. This is the only place with access to `section.blocks`.

**No separate `{% doc %}` needed** -- this will be inline in the FAQ section, not a separate snippet. See "Files to Modify" section below for details.

---

### 7. `snippets/seo-resource-hints.liquid` (NEW)

Preconnect and DNS-prefetch hints for known external origins.

**Output:**
```html
<link rel="preconnect" href="https://cdn.shopify.com" crossorigin>
<link rel="preconnect" href="https://fonts.shopifycdn.com" crossorigin>
<link rel="dns-prefetch" href="https://cdn.shopify.com">
<link rel="dns-prefetch" href="https://fonts.shopifycdn.com">
```

**`{% doc %}` header params:**
- None

---

## Files to Modify

### 8. `layout/theme.liquid`

**Changes (in order within `<head>`):**

After line 19 (`<link rel="canonical">`), add:
```liquid
{%- render 'seo-meta-tags' -%}
{%- render 'seo-resource-hints' -%}
```

Before the closing `</head>` (after font_face style tags, around line 39), add:
```liquid
{%- render 'seo-jsonld-organization' -%}

{%- if template.name == 'product' -%}
  {%- render 'seo-jsonld-product', product: product -%}
  {%- render 'seo-jsonld-breadcrumb' -%}
{%- endif -%}

{%- if template.name == 'collection' -%}
  {%- render 'seo-jsonld-collection', collection: collection -%}
  {%- render 'seo-jsonld-breadcrumb' -%}
{%- endif -%}
```

**Rationale:** JSON-LD scripts are valid anywhere in the document, but placing them in `<head>` keeps them together and ensures they are parsed before the body. The Organization and WebSite schemas are sitewide. Product, Collection, and Breadcrumb schemas are conditional on template type.

---

### 9. `sections/faq-section.liquid`

**Change:** Add FAQPage JSON-LD inline, between the closing `</section>` tag (line 114) and `{% stylesheet %}` (line 116).

Insert:
```liquid
{%- comment -%} FAQPage structured data {%- endcomment -%}
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {%- for block in section.blocks -%}
      {%- if block.type == 'faq_item' -%}
        {%- unless forloop.first -%},{%- endunless -%}
        {
          "@type": "Question",
          "name": {{ block.settings.question | json }},
          "acceptedAnswer": {
            "@type": "Answer",
            "text": {{ block.settings.answer | strip_html | json }}
          }
        }
      {%- endif -%}
    {%- endfor -%}
  ]
}
</script>
```

**Rationale:** The FAQ blocks are only accessible inside the section. This cannot be a snippet rendered from layout because snippets do not have access to `section.blocks`. Placing the `<script type="application/ld+json">` between `</section>` and `{% stylesheet %}` keeps it clean and avoids nesting issues.

---

### 10. `config/settings_schema.json`

**Add a new "SEO" settings group** after the existing "Dark mode" group (before the "Cart" group). This goes between the object at index 7 and index 8 in the array.

```json
{
  "name": "t:settings.seo.name",
  "settings": [
    {
      "type": "header",
      "content": "t:settings.seo.organization_header.content"
    },
    {
      "id": "seo_organization_logo",
      "type": "image_picker",
      "label": "t:settings.seo.seo_organization_logo.label",
      "info": "t:settings.seo.seo_organization_logo.info"
    },
    {
      "type": "header",
      "content": "t:settings.seo.social_sharing_header.content"
    },
    {
      "id": "seo_default_image",
      "type": "image_picker",
      "label": "t:settings.seo.seo_default_image.label",
      "info": "t:settings.seo.seo_default_image.info"
    },
    {
      "id": "seo_twitter_handle",
      "type": "text",
      "label": "t:settings.seo.seo_twitter_handle.label",
      "placeholder": "@yourhandle",
      "info": "t:settings.seo.seo_twitter_handle.info"
    },
    {
      "type": "header",
      "content": "t:settings.seo.advanced_header.content"
    },
    {
      "id": "seo_noindex_paginated",
      "type": "checkbox",
      "label": "t:settings.seo.seo_noindex_paginated.label",
      "info": "t:settings.seo.seo_noindex_paginated.info",
      "default": true
    },
    {
      "id": "seo_noindex_filtered",
      "type": "checkbox",
      "label": "t:settings.seo.seo_noindex_filtered.label",
      "info": "t:settings.seo.seo_noindex_filtered.info",
      "default": true
    }
  ]
}
```

---

### 11. `locales/en.default.json`

**Add new top-level key `"seo"`:**

```json
{
  "seo": {
    "og_product_type": "product",
    "og_website_type": "website",
    "breadcrumb_home": "Home"
  }
}
```

Note: Most SEO output uses Shopify object data directly (product.title, shop.name, etc.) and does not need translation keys. The breadcrumb "Home" label reuses `general.home` which already exists. The keys above are for edge cases only.

Actually, `general.home` already exists at value "Home", so we only need the new keys if the "Home" label needs to differ in breadcrumb structured data. Since it does not, we can reuse `general.home` and skip adding `seo.breadcrumb_home`.

**Revised -- only add:**

```json
{
  "seo": {
    "default_description": "Shop premium fitness gear and programs at {{ shop_name }}"
  }
}
```

This is used as a fallback when `page_description` is blank.

---

### 12. `locales/en.default.schema.json`

**Add under the `"settings"` object:**

```json
"seo": {
  "name": "SEO",
  "organization_header": { "content": "Organization schema" },
  "seo_organization_logo": {
    "label": "Organization logo",
    "info": "Used in Google search results. Falls back to header logo if empty."
  },
  "social_sharing_header": { "content": "Social sharing" },
  "seo_default_image": {
    "label": "Default sharing image",
    "info": "Fallback image for Open Graph / Twitter cards when no page-specific image exists. Recommended: 1200x630px."
  },
  "seo_twitter_handle": {
    "label": "Twitter / X handle",
    "info": "Include the @ symbol, e.g. @zenithfitness"
  },
  "advanced_header": { "content": "Advanced" },
  "seo_noindex_paginated": {
    "label": "Noindex paginated pages",
    "info": "Prevents search engines from indexing page 2, 3, etc. of collections."
  },
  "seo_noindex_filtered": {
    "label": "Noindex filtered collections",
    "info": "Prevents indexing of tag-filtered collection URLs to avoid duplicate content."
  }
}
```

---

## Complete File Inventory

| # | File | Action | Purpose |
|---|---|---|---|
| 1 | `snippets/seo-meta-tags.liquid` | CREATE | Open Graph, Twitter Card, robots meta |
| 2 | `snippets/seo-jsonld-organization.liquid` | CREATE | Organization + WebSite + SearchAction schema |
| 3 | `snippets/seo-jsonld-product.liquid` | CREATE | Product schema with Offer(s), ratings, images |
| 4 | `snippets/seo-jsonld-breadcrumb.liquid` | CREATE | BreadcrumbList schema |
| 5 | `snippets/seo-jsonld-collection.liquid` | CREATE | CollectionPage + ItemList schema |
| 6 | `snippets/seo-resource-hints.liquid` | CREATE | Preconnect / dns-prefetch hints |
| 7 | `layout/theme.liquid` | MODIFY | Render all SEO snippets |
| 8 | `sections/faq-section.liquid` | MODIFY | Add inline FAQPage JSON-LD |
| 9 | `config/settings_schema.json` | MODIFY | Add SEO settings group |
| 10 | `locales/en.default.json` | MODIFY | Add `seo` key with fallback description |
| 11 | `locales/en.default.schema.json` | MODIFY | Add `settings.seo` editor labels |

---

## Build Order

| Step | File(s) | Reason |
|---|---|---|
| 1 | `config/settings_schema.json` | Foundation -- settings must exist before snippets reference them |
| 2 | `locales/en.default.json`, `locales/en.default.schema.json` | Translation keys must exist before schema labels reference them |
| 3 | `snippets/seo-resource-hints.liquid` | No dependencies, simplest snippet |
| 4 | `snippets/seo-meta-tags.liquid` | Depends on settings (step 1) |
| 5 | `snippets/seo-jsonld-organization.liquid` | Depends on settings (step 1), social URLs |
| 6 | `snippets/seo-jsonld-breadcrumb.liquid` | No dependencies beyond Shopify globals |
| 7 | `snippets/seo-jsonld-product.liquid` | Depends on product object knowledge |
| 8 | `snippets/seo-jsonld-collection.liquid` | Depends on collection object knowledge |
| 9 | `sections/faq-section.liquid` | Modify existing section to add inline FAQPage schema |
| 10 | `layout/theme.liquid` | Wire everything together -- must be last |
| 11 | **Validate** | Run `validate_theme` via Shopify Dev MCP |

---

## Translation Keys

### `locales/en.default.json` additions

```
seo
  default_description    "Shop premium fitness gear and programs at {{ shop_name }}"
```

### `locales/en.default.schema.json` additions

```
settings
  seo
    name                           "SEO"
    organization_header.content    "Organization schema"
    seo_organization_logo.label    "Organization logo"
    seo_organization_logo.info     "Used in Google search results. Falls back to header logo if empty."
    social_sharing_header.content  "Social sharing"
    seo_default_image.label        "Default sharing image"
    seo_default_image.info         "Fallback image for Open Graph / Twitter cards..."
    seo_twitter_handle.label       "Twitter / X handle"
    seo_twitter_handle.info        "Include the @ symbol..."
    advanced_header.content        "Advanced"
    seo_noindex_paginated.label    "Noindex paginated pages"
    seo_noindex_paginated.info     "Prevents search engines from indexing page 2, 3..."
    seo_noindex_filtered.label     "Noindex filtered collections"
    seo_noindex_filtered.info      "Prevents indexing of tag-filtered collection URLs..."
```

---

## Detailed Snippet Specifications

### `snippets/seo-meta-tags.liquid`

```
{% doc %}
  Outputs Open Graph, Twitter Card, and robots meta tags.
  Rendered in <head> via layout/theme.liquid.
  No parameters -- uses global Shopify objects.

  @example
    {%- render 'seo-meta-tags' -%}
{% enddoc %}
```

**Conditional logic summary:**

1. **og:type**: `product` when `template.name == 'product'`, else `website`
2. **og:image resolution order**:
   - Product page: `product.featured_image | image_url: width: 1200`
   - Collection page: `collection.image | image_url: width: 1200`
   - All other pages: `settings.seo_default_image | image_url: width: 1200`
   - If none available: omit og:image entirely
3. **Product-specific OG tags**: Only on product template -- `og:price:amount`, `og:price:currency`, `product:availability`
4. **Robots noindex**: Applied when `settings.seo_noindex_paginated` is true AND `current_page > 1`, OR when `settings.seo_noindex_filtered` is true AND `current_tags` is not empty
5. **Twitter handle**: Only output `twitter:site` if `settings.seo_twitter_handle` is not blank

### `snippets/seo-jsonld-product.liquid`

```
{% doc %}
  Outputs Product structured data as JSON-LD.

  @param {product} product - The product object (required)

  @example
    {%- render 'seo-jsonld-product', product: product -%}
{% enddoc %}
```

**Key implementation details:**

- Use `| json` filter for all string values to handle quotes/special chars
- Use `| money_without_currency | remove: ','` for price values
- Map availability: `product.available` -> `https://schema.org/InStock` / `https://schema.org/OutOfStock`
- For single-variant products (`product.has_only_default_variant`), use `"@type": "Offer"` with the single variant's price
- For multi-variant products, use `"@type": "AggregateOffer"` with `lowPrice`/`highPrice`
- Include `aggregateRating` only when `product.metafields.reviews.rating` exists. Use `product.metafields.reviews.rating.value` for `ratingValue` and `product.metafields.reviews.rating_count` for `reviewCount`
- Include `"image"` as an array of up to 10 media image URLs at 1200px width
- Include `"sku"` from the first available variant
- Include `"brand"` from `product.vendor` if not blank

### `snippets/seo-jsonld-collection.liquid`

```
{% doc %}
  Outputs CollectionPage + ItemList structured data as JSON-LD.

  @param {collection} collection - The collection object (required)

  @example
    {%- render 'seo-jsonld-collection', collection: collection -%}
{% enddoc %}
```

**Key details:**
- Loop over `collection.products` (limited to current page, typically 12-50)
- Each product becomes a `ListItem` with `position`, `url`, `name`, and optional `image`
- Use `forloop.index` for position numbering

### `snippets/seo-jsonld-breadcrumb.liquid`

```
{% doc %}
  Outputs BreadcrumbList structured data as JSON-LD.
  Automatically detects product vs collection context.

  @example
    {%- render 'seo-jsonld-breadcrumb' -%}
{% enddoc %}
```

**Key details:**
- Position 1 is always Home with `{{ shop.url }}`
- On collection pages: Position 2 is `collection.title` (no `item` URL -- current page)
- On product pages: Position 2 is `collection.title` with `collection.url` (if `collection` exists), Position 3 is `product.title` (no `item` URL -- current page)
- All URLs must be fully qualified (`{{ shop.url }}{{ path }}`)

---

## Performance & Technical SEO Notes

### Image lazy loading audit (current state)
The theme already has good lazy loading coverage:
- Hero banner: `fetchpriority="high"` on main image (correct -- above the fold)
- Header logo: `fetchpriority="high"` (correct)
- Product detail: First image `loading="eager"`, subsequent `loading="lazy"` (correct)
- All other sections: `loading="lazy"` (correct)
- No changes needed.

### Critical CSS review
- Single `critical.css` file loaded in `<head>` -- good
- Per-component CSS via `{% stylesheet %}` -- good
- Shared styles (.btn, etc.) in critical.css -- follows conventions
- No changes needed.

### Preconnect strategy
- `cdn.shopify.com` -- all product images, assets
- `fonts.shopifycdn.com` -- Shopify-hosted fonts (Syne, Inter, Source Serif 4)
- Both get `preconnect` (establish full connection) and `dns-prefetch` (fallback for older browsers)

### What we intentionally omit
- **Article/BlogPosting schema**: No blog or article templates exist in this theme
- **Review schema as standalone**: Covered within Product schema via `aggregateRating`
- **Breadcrumb on non-product/collection pages**: Low SEO value; product and collection pages are the priority
- **hreflang tags**: Single-language theme (English only)
- **AMP**: Not applicable to Shopify themes

---

## Validation Checklist

After implementation, verify:

1. [ ] Run `validate_theme` via Shopify Dev MCP -- zero errors
2. [ ] Test Product page JSON-LD with Google Rich Results Test
3. [ ] Test Collection page JSON-LD with Google Rich Results Test
4. [ ] Test FAQ page JSON-LD with Google Rich Results Test
5. [ ] Verify Open Graph tags with Facebook Sharing Debugger
6. [ ] Verify Twitter Card with Twitter Card Validator
7. [ ] Confirm robots noindex on page 2+ of collections
8. [ ] Confirm robots noindex on tag-filtered collection URLs
9. [ ] Confirm no JSON-LD syntax errors (valid JSON in all `<script type="application/ld+json">` blocks)
10. [ ] Confirm preconnect hints appear before any resource requests in `<head>`
11. [ ] Confirm all user-facing text uses `| t` translation pattern
12. [ ] Confirm all snippet files have `{% doc %}` headers
