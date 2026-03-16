# ADR-0001: Project Setup and Multi-Theme Architecture

**Status:** Accepted
**Date:** 2026-03-16

## Context

We are building a Shopify e-commerce project with 3 distinct themes targeting different merchant needs. Each theme will be a standalone Shopify theme using Liquid templating, following Shopify's Online Store 2.0 architecture.

## Decision

### Project Structure

```
shopify-test/
├── ADR/                          # Architecture Decision Records
├── themes/
│   ├── starter-minimal/          # Theme 1: Clean, minimal storefront
│   ├── starter-bold/             # Theme 2: Bold, image-heavy brand theme
│   └── starter-developer/        # Theme 3: Developer-focused, extensible theme
└── docs/                         # Shared documentation
```

Each theme follows Shopify's standard directory structure:

```
theme-name/
├── assets/          # Static assets (critical.css only; prefer {% stylesheet %})
├── blocks/          # Reusable, nestable, customizable components
├── config/          # settings_schema.json + settings_data.json
├── layout/          # theme.liquid (main layout)
├── locales/         # en.default.json, en.default.schema.json
├── sections/        # Full-width modular page components
├── snippets/        # Reusable Liquid fragments
└── templates/       # JSON templates (index, product, collection, etc.)
```

### Three Themes

| Theme | Target Merchant | Key Characteristics |
|-------|----------------|---------------------|
| **starter-minimal** | Small boutiques, artisans | Clean typography, whitespace-focused, fast loading, minimal JS |
| **starter-bold** | Fashion, lifestyle brands | Hero-heavy, large imagery, bold colors, animated transitions |
| **starter-developer** | Developers / agencies | Highly modular, well-documented blocks, extensible architecture |

### Technology Stack

- **Templating:** Liquid (Shopify's templating language)
- **Styling:** CSS via `{% stylesheet %}` tags per component (no external CSS frameworks)
- **JavaScript:** Vanilla JS via `{% javascript %}` tags per component (no frameworks)
- **Theme Management:** Shopify CLI 3.0
- **Validation:** Shopify Dev MCP `validate_theme` tool
- **Localization:** English default with i18n-ready structure (`{{ 'key' | t }}`)

### Key Architectural Principles

1. **Online Store 2.0 compliant** - JSON templates, sections everywhere, blocks with `{% schema %}`
2. **All user-facing text uses translation keys** - `{{ 'key' | t }}` pattern
3. **CSS/JS per component** - Using `{% stylesheet %}` and `{% javascript %}` tags, not global asset files
4. **Accessible** - WCAG 2.1, semantic HTML, keyboard navigation
5. **Mobile-first** - Responsive design with mobile column settings
6. **No external dependencies** - No CSS/JS libraries, everything from scratch
7. **LiquidDoc on all snippets and static blocks** - `{% doc %}` headers for documentation

## Prerequisites (External Work Required)

### Step 1: Shopify Partner Account (FREE)
1. Go to **partners.shopify.com** and sign up for a free Partner account
2. This gives you access to the Partner Dashboard where you manage apps, themes, and stores

### Step 2: Development Store (FREE)
1. In the Partner Dashboard, go to **Stores > Add store > Create development store**
2. Choose "Create a store to test and build"
3. Pick a store name (e.g., `my-theme-dev-store`)
4. Select a development plan - this is free and has no time limit
5. You will need **one dev store per theme** for simultaneous testing, or reuse one store and swap themes

### Step 3: Install Shopify CLI
```bash
npm i -g @shopify/cli@latest
```
Verify with:
```bash
shopify version
```

### Step 4: Authenticate CLI
```bash
shopify auth login --store=your-dev-store.myshopify.com
```
This opens a browser for OAuth. No API keys needed for theme development.

### Step 5: (Optional) Connect Dev MCP Server
Already configured in this project. The MCP server at `@shopify/dev-mcp@latest` provides:
- `validate_theme` - Validates Liquid code
- `introspect_graphql_schema` - For any app extensions later
- `search_docs_chunks` - Search Shopify docs

## First Steps (Implementation Order)

1. **Install Shopify CLI** (see prerequisites above)
2. **Create Partner account + dev store** (see prerequisites above)
3. **Scaffold theme 1 (starter-minimal)** using `shopify theme init`
4. Build shared snippets (image, button, icon) reusable across themes
5. Build core sections (header, footer, hero, product-grid, featured-collection)
6. Build core blocks (text, image, group, heading)
7. Set up `config/settings_schema.json` for global theme settings (colors, typography)
8. Create JSON templates (index, product, collection, cart, page)
9. Set up `locales/en.default.json` with all translation keys
10. Validate with `validate_theme` and test on dev store with `shopify theme dev`
11. Repeat steps 3-10 for themes 2 and 3, customizing design language

## Consequences

- Three separate theme codebases means some code duplication, but each theme can evolve independently
- Shared snippets can be copied between themes but are not auto-synced
- Each theme can be submitted to the Shopify Theme Store or used privately
- No vendor lock-in beyond Shopify's platform itself
