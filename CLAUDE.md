# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

E-commerce project with 3 completed Shopify OS 2.0 themes under `themes/`, each targeting a different merchant persona. All themes are built with Liquid and follow Shopify Online Store 2.0 architecture.

### Dev Store
- URL: https://devstore-2795063.myshopify.com
- Store password: autsow

### Themes

| Theme | Directory | Target | Shopify ID | Status |
|---|---|---|---|---|
| BrewMaster | `themes/starter-minimal` | Small boutiques | #161419264240 | Published (live) |
| ArtVault | `themes/starter-bold` | Art/creative studios | #161425916144 | Unpublished |
| CodeCraft Academy | `themes/starter-developer` | Agencies, SaaS, courses | #161428340976 | Unpublished |
| Patina | `themes/starter-luxury` | Fine jewelry, luxury | #161439547632 | Unpublished |
| PawHaven | `themes/starter-pets` | Pet supplies | #161442234608 | Unpublished |
| Zenith | `themes/starter-fitness` | Fitness & wellness | #161442988272 | Unpublished |

**starter-minimal (BrewMaster)** — 76 files. Amber (#d97706) primary, white background, Helvetica fonts. Clean typography, whitespace-focused, fast loading. Plan: `PLANS/themes/1/PLAN-0001-brewmaster-shopify-theme.md`. Figma source: file IrYnqvFhgUXJAmw10tmxoe.

**starter-bold (ArtVault)** — 69 files. Forest green (#2D5A3D) + warm wood (#C8956C), Playfair Display serif + Inter, dense editorial layouts. Features: glassmorphism header, marquee strip, editorial split, social feed, testimonials, video background hero. Plan: `PLANS/themes/2/PLAN-0002-artvault-shopify-theme.md`. Figma: https://www.figma.com/design/g2fn6NTKAz2GtmEyNYxMUE

**starter-developer (CodeCraft Academy)** — 57 files. Dark terminal (#0B1120) + cyan (#06B6D4), JetBrains Mono/Space Grotesk, retro hacker aesthetic. Features: command bar header, terminal window component, dark mode toggle, comparison pricing table, animated counters, gradient text, tech stack badges. Design HTML: `PLANS/themes/3/design-developer-theme.html`. Figma: https://www.figma.com/design/0SaGse6Scy9l1hehrxojCK

**starter-luxury (Patina)** — 52 files. Warm terracotta (#C2734C) + cream (#FAF7F2), Cormorant Garamond italic + Lato, retro/vintage aesthetic. Features: transparent header overlay, hover-to-reveal product cards, asymmetric lookbook gallery, editorial brand story, testimonials, newsletter CTA. Sharp edges (0px radius), no shadows, thin borders.

**starter-pets (PawHaven)** — 52 files. Forest emerald (#059669) + warm sand (#F5F0E8), Nunito + Quicksand, earthy/natural aesthetic. Features: emerald announcement topbar, category carousel with circular cards, mosaic product grid, rounded cards with soft shadows (12px radius), scroll-reveal animations, mini cart dropdown, testimonial carousel, newsletter CTA.

**starter-fitness (Zenith)** — 64 files. Soft black (#0A0A0A) + electric teal (#00D4AA) + coral (#FF6B6B), Syne/Inter/Source Serif 4 3-font system, ultra-premium cinematic editorial. Features: quick-view modal, product comparison, countdown timer, recently viewed (localStorage), swipeable product cards, tabbed gallery, multi-step cart, mega menu with product previews, bottom tab bar (mobile), dark/light mode toggle, FAQ with search, program showcase with pricing, animated stat counters, scroll-snap cinematic layout, parallax effects, glow animations.

## Development Commands

```bash
# Preview a theme locally (run from within a theme directory)
cd themes/starter-minimal
shopify theme dev --store=devstore-2795063.myshopify.com

# Push theme to dev store (safe push, preserves existing files)
shopify theme push --store=devstore-2795063.myshopify.com --nodelete

# Push to the live/published theme (BrewMaster)
shopify theme push --store=devstore-2795063.myshopify.com --theme 161419264240 --nodelete --allow-live

# Pull latest theme state from store (always pull before pushing)
shopify theme pull --store=devstore-2795063.myshopify.com

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

## Coding Conventions

### Liquid
- All user-facing text MUST use translation keys: `{{ 'key' | t }}`
- Use `{% stylesheet %}` for CSS per component, not global asset files
- Use `<script>` tags for JavaScript (NOT `{% javascript %}` — see lessons learned)
- All snippets and statically-rendered blocks MUST have `{% doc %}` headers
- Sections and blocks MUST include `{% schema %}` for theme editor customization
- Use sentence case for all user-facing text
- Use nested `if` conditions instead of complex logical operators (no parentheses in Liquid)
- Never use legacy resource-based settings; access objects directly
- Use `{% content_for 'blocks' %}` for standalone blocks from `blocks/` directory
- Section schema must use `"type": "@theme"` for standalone block references
- Pre-assign translated strings to variables before passing to `{% render %}` tags
- Snippets use raw `<style>` tags (not `{% stylesheet %}`)
- Wrap `font_face` filter output in its own `<style>` tag to prevent raw CSS text rendering

### CSS
- Single CSS property settings: use CSS variables (`--gap`, `--color`, etc.)
- Multiple CSS property settings: use CSS classes
- Mobile-first responsive design
- No external CSS libraries
- Move shared CSS (like `.btn`) to `critical.css` to avoid injection issues across components

### JavaScript
- Vanilla JS only, no frameworks or libraries
- Use `<script>` tags, NOT `{% javascript %}` (the Liquid tag does not render output)

### Accessibility
- WCAG 2.1 compliance
- Semantic HTML (`<details>`, `<summary>`, `<dialog>`, etc.)
- Keyboard navigation and screen reader support

### Localization
- English default (`locales/en.default.json`)
- Hierarchical keys, max 3 levels deep, snake_case
- Use interpolation for dynamic content: `{{ 'key' | t: var: value }}`

## Key Lessons Learned

These are hard-won findings from building all 3 themes:

1. **Use `<script>` tags, not `{% javascript %}`** — The `{% javascript %}` Liquid tag does not render output; use standard `<script>` tags instead
2. **Use `{% content_for 'blocks' %}` for standalone blocks** — Required when using block files from the `blocks/` directory
3. **Section schema `"type": "@theme"`** — Required in section schema for standalone block references
4. **Shared CSS goes in `critical.css`** — Move shared styles (like `.btn`) to `critical.css` to avoid injection/ordering issues
5. **Pre-assign translations before `{% render %}`** — Translated strings must be assigned to variables before passing them as parameters to `{% render %}` tags
6. **Snippets use raw `<style>` tags** — Snippets cannot use `{% stylesheet %}`; use plain `<style>` tags instead
7. **Wrap `font_face` in `<style>`** — The `font_face` filter output must be wrapped in its own `<style>` tag or it renders as raw text
8. **Use `--nodelete --allow-live` for live pushes** — Prevents deleting files and allows pushing to a published theme
9. **Always pull before pushing** — Pull theme editor changes before pushing to avoid overwriting merchant customizations
10. **JetBrains Mono is not in Shopify's font picker** — Use `anonymous_pro_n4` as the monospace default fallback

## MCP Servers
- **Shopify Dev MCP** (`@shopify/dev-mcp@latest`): `validate_theme`, `search_docs_chunks`, `learn_shopify_api` (api: "liquid"), `introspect_graphql_schema`
- **Figma MCP**: `get_design_context`, `get_screenshot`, `generate_figma_design` — used for translating Figma designs to theme plans
- **Playwright MCP**: Browser testing for visual verification
- **NotebookLM MCP**: Project knowledge base at notebook `e4d10d43-52c9-48e3-bc72-6183ecc802b2`

## Git Workflow
- Main branch: `main`
- Feature branches per theme: `feature/brewmaster-theme`, `feature/starter-bold-theme`, `feature/starter-developer-theme`
- Themes 1 (BrewMaster) and 2 (ArtVault) are merged to `main`
- Theme 3 (CodeCraft Academy) is on `feature/starter-developer-theme`
- Commit messages: concise, focused on "why"
- One logical change per commit
- ADR documents for architectural decisions in `ADR/` folder
