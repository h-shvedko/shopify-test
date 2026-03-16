# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
E-commerce project with 3 Shopify themes (starter-minimal, starter-bold, starter-developer) built with Liquid, following Shopify Online Store 2.0 architecture. Each theme is a standalone theme under `themes/`.

## Development Commands

```bash
# Preview a theme locally (run from within a theme directory)
cd themes/starter-minimal
shopify theme dev --store=your-dev-store.myshopify.com

# Push theme to dev store
shopify theme push --store=your-dev-store.myshopify.com

# Pull latest theme state from store
shopify theme pull --store=your-dev-store.myshopify.com

# Install/update Shopify CLI
npm i -g @shopify/cli@latest
shopify version
```

## Validation
- ALWAYS run `validate_theme` via the Shopify Dev MCP after generating or modifying theme files
- Fix all validation errors before committing

## Architecture

Each theme is independent with the standard Shopify OS 2.0 structure:
- **templates/** — JSON templates that compose sections (index, product, collection, cart, page)
- **sections/** — Full-width page modules with `{% schema %}` for theme editor settings
- **blocks/** — Reusable nestable components with `{% schema %}`; used inside sections
- **snippets/** — Reusable Liquid fragments; MUST have `{% doc %}` headers
- **layout/theme.liquid** — Main layout wrapper
- **config/** — `settings_schema.json` (global theme settings) + `settings_data.json` (current values)
- **locales/** — `en.default.json` (storefront strings) + `en.default.schema.json` (editor strings)
- **assets/** — Only `critical.css`; prefer `{% stylesheet %}` per component

Three themes target different merchants:
- **starter-minimal** — Small boutiques; clean typography, whitespace-focused, fast loading
- **starter-bold** — Fashion/lifestyle; hero-heavy, large imagery, bold colors
- **starter-developer** — Agencies; highly modular, well-documented, extensible

## Coding Conventions

### Liquid
- All user-facing text MUST use translation keys: `{{ 'key' | t }}`
- Use `{% stylesheet %}` and `{% javascript %}` tags per component, not global asset files
- All snippets and statically-rendered blocks MUST have `{% doc %}` headers
- Sections and blocks MUST include `{% schema %}` for theme editor customization
- Use sentence case for all user-facing text
- Use nested `if` conditions instead of complex logical operators (no parentheses in Liquid)
- Never use legacy resource-based settings; access objects directly

### CSS
- Single CSS property settings: use CSS variables (`--gap`, `--color`, etc.)
- Multiple CSS property settings: use CSS classes
- Mobile-first responsive design
- No external CSS libraries

### JavaScript
- Vanilla JS only, no frameworks or libraries
- One `{% javascript %}` tag per section/block/snippet

### Accessibility
- WCAG 2.1 compliance
- Semantic HTML (`<details>`, `<summary>`, `<dialog>`, etc.)
- Keyboard navigation and screen reader support

### Localization
- English default (`locales/en.default.json`)
- Hierarchical keys, max 3 levels deep, snake_case
- Use interpolation for dynamic content: `{{ 'key' | t: var: value }}`

## MCP Servers
- **Shopify Dev MCP** (`@shopify/dev-mcp@latest`): Use `learn_shopify_api` (api: "liquid") first when learning Liquid APIs, then `validate_theme`, `search_docs_chunks`, `introspect_graphql_schema`
- **NotebookLM MCP**: Project knowledge base at notebook `e4d10d43-52c9-48e3-bc72-6183ecc802b2`

## Git Workflow
- Commit messages: concise, focused on "why"
- One logical change per commit
- ADR documents for architectural decisions in `ADR/` folder
