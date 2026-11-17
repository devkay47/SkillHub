# shopify-theme-validator

Validate a Shopify theme before upload and catch the defects Shopify's own theme-check misses.

## The problem

Shopify's own validator, theme-check, does not catch everything that breaks an upload. The defects it misses are the ones that cost hours:

1. A block name over 25 characters. Shopify rejects it on save, and a long block name once made an upload silently drop two files, with no error shown.
2. A duplicate setting `id` inside a section or block settings array. theme-check only checks `settings_schema.json`.
3. A hardcoded price, such as $49.99, written literally in code instead of going through the money filter.
4. Bare `structured_data` that renders as visible text on the page.
5. Placeholder copy left in the theme, such as "Sample product" or "$00.00".

This skill teaches the agent the hard rules learned from real theme work, where each rule is a failure that already happened once. They are the same rules that caught the v1.2.1 upload defect, where two long block names dropped `templates/product.json` and `sections/main-product.liquid`.

## What is in the free skill

- `shopify-theme-validator/SKILL.md`: the hard rules with the exact Shopify error strings, what theme-check catches and what it misses, how to run the validator, the upload workflow, a worked hero banner example, and a troubleshooting table.
- `shopify-theme-validator/references/theme-check-rules.md`: the full rule reference, with the sources for every rule (theme-check 3.24.0, the project's own validator, the v1.2.1 defect).
- `shopify-theme-validator/INSTALL.md`: install steps for Claude Code, Codex CLI, Cursor, and Claude.ai.
- `shopify-theme-validator/examples/hero-banner.liquid`: the worked example from SKILL.md, a clean hero section that passes the validator.
- `shopify-theme-validator/examples/home.json`: a template that uses the hero section.
- `shopify-theme-validator/examples/validator-checklist.md`: the pre-delivery checklist.

## Install

```bash
mkdir -p ~/.claude/skills
cp -r shopify-theme-validator ~/.claude/skills/
```

See `shopify-theme-validator/INSTALL.md` for Codex, Cursor, and Claude.ai.

## Premium pack

The premium pack adds three things the free skill does not have:

- `validate.py`: a script that checks a theme before upload. It catches block and section names over 25 characters, duplicate setting ids inside any settings array, setting ids named `size`, `first`, or `last` that collide with Liquid's built-in filters, deprecated image filters (`img_url` and family), bare structured_data, placeholder copy, hardcoded prices, the default filter-chain trap, layout files missing content_for objects, JavaScript syntax errors, template section references that do not exist, block types a template uses but the section does not define, and stale block_order entries. Python 3, no dependencies. `validate.py --fix` repairs duplicate setting ids and rebinds the Liquid references, writing new files under `fixed/` and never overwriting the input.
- A reference library: six fresh generic files in valid theme form, a hero banner section, an image with text section with two block types, a template that uses both, a settings schema, a mega menu section, and a footer section with a `locales` object.
- A push-to-unpublished checklist: the steps to push a theme to an unpublished theme, preview it on a real store, check the routes, and publish only after the preview passes. Plus a CLI rescue toolkit: the steps when the Shopify CLI misbehaves, ghost dev servers, stale caches, and a forced fresh login.

Get the premium pack at https://websmartools.com/shopify-theme-validator/ for $15, one-time payment, single-user licence.

## Licence

The free skill is MIT licensed. See LICENSE. The premium pack has its own single-user licence and is not open source.
