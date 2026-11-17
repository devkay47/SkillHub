# shopify-theme-validator

> Validate a Shopify theme before upload and catch the defects Shopify's own theme-check misses. Use when the user asks to check, validate, or fix a Shopify theme, when an upload drops files, when a section or block fails to save, or before pushing a theme to an unpublished theme for preview.

**Status:** Free. The validator script (`validate.py`) and the premium checklist are in the premium pack; this skill documents the checks and how to run them.

## What's inside

- `shopify-theme-validator/SKILL.md` — the skill itself: the hard rules, what theme-check catches and misses, how to run `validate.py`, a worked hero-banner example, and troubleshooting.
- `shopify-theme-validator/examples/hero-banner.liquid` — a clean hero section the validator passes.
- `shopify-theme-validator/references/` — the full check list and the validation commands.
- `INSTALL.md` — how to install the skill in Claude Code.
- `CHANGELOG.md` — version history.

## Install

Copy the skill folder into your Claude Code skills directory:

```bash
cp -r shopify-theme-validator ~/.claude/skills/
```

Or install from the SkillHub registry once listed.

## Usage

Ask Claude Code to validate a theme:

> "Validate my Shopify theme at ./my-theme"

The skill runs `validate.py` over the theme folder and reports errors (must fix before upload) and warnings (check by hand).

## Checks covered

- Section and block names are 25 characters or fewer.
- Setting ids are unique inside each settings array.
- Deprecated image filters (`img_url`, `article_img_url`, `collection_img_url`, `product_img_url`).
- Templates reference sections that exist, and use block types the section defines.
- Layouts contain `{{ content_for_header }}` and `{{ content_for_layout }}`.
- Bare `structured_data` output, placeholder copy, and hardcoded prices.
- JavaScript syntax, checked with `node --check` when node is present.

## License

MIT. See [LICENSE](LICENSE).
