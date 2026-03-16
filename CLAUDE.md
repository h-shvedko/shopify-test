# Shopify Multi-Theme E-Commerce Project

## Project Overview
E-commerce project with 3 Shopify themes (starter-minimal, starter-bold, starter-developer) built with Liquid, following Shopify Online Store 2.0 architecture.

## Tech Stack
- **Templating:** Shopify Liquid
- **Styling:** CSS via `{% stylesheet %}` tags (no external frameworks)
- **JavaScript:** Vanilla JS via `{% javascript %}` tags (no frameworks)
- **CLI:** Shopify CLI 3.0 (`shopify theme dev`, `shopify theme push`)
- **Validation:** Shopify Dev MCP `validate_theme` tool

## Project Structure
```
shopify-test/
├── ADR/                          # Architecture Decision Records
├── themes/
│   ├── starter-minimal/          # Clean, minimal storefront
│   ├── starter-bold/             # Bold, image-heavy brand theme
│   └── starter-developer/        # Developer-focused, extensible theme
└── CLAUDE.md
```

Each theme follows:
```
theme-name/
├── assets/       # Only critical.css; prefer {% stylesheet %} per component
├── blocks/       # Reusable nestable components with {% schema %}
├── config/       # settings_schema.json + settings_data.json
├── layout/       # theme.liquid
├── locales/      # en.default.json, en.default.schema.json
├── sections/     # Full-width modules with {% schema %}
├── snippets/     # Reusable fragments with {% doc %} headers
└── templates/    # JSON templates
```

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
- Keyboard navigation support
- Screen reader support

### Localization
- English default (`locales/en.default.json`)
- Hierarchical keys, max 3 levels deep, snake_case
- Use interpolation for dynamic content: `{{ 'key' | t: var: value }}`

## Validation
- ALWAYS run `validate_theme` via the Shopify Dev MCP after generating or modifying theme files
- Fix all validation errors before committing

## MCP Servers
- **Shopify Dev MCP** (`@shopify/dev-mcp@latest`): Use `learn_shopify_api` (api: "liquid") first, then `validate_theme`, `search_docs_chunks`, `introspect_graphql_schema`
- **NotebookLM MCP**: Project knowledge base at notebook `e4d10d43-52c9-48e3-bc72-6183ecc802b2`

## Git Workflow
- Commit messages: concise, focused on "why"
- One logical change per commit
- ADR documents for architectural decisions in `ADR/` folder
