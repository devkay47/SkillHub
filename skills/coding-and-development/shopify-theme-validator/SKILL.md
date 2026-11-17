---
name: shopify-theme-validator
description: Validate a Shopify theme before upload and catch the defects Shopify's own theme-check misses. Use when the user asks to check, validate, or fix a Shopify theme, when an upload drops files, when a section or block fails to save, or before pushing a theme to an unpublished theme for preview.
---

# Shopify Theme Validator

A narrow validator for Shopify themes. It checks a theme's section and block schemas, template JSON, and Liquid files for the defects that break uploads and saves, and reports them before anything goes live. It does not teach theme building. For building and editing themes, use the shopify-liquid skill. This skill only validates.

## When to Use

- Validating a theme before upload, after any schema change (a new section, a new block, a renamed setting).
- When Shopify drops files on upload, for example a template that was in the ZIP is missing from the installed theme.
- When a save fails with an `Invalid block` or schema error.
- Before pushing a theme to an unpublished theme for preview.

## The Hard Rules

These rules were learned from real theme work, where each one is a failure that already happened once. The validator checks all of them.

1. **Block `name` is 25 characters max.** Shopify rejects a longer name on save with `Invalid block '<type>': name is too long (max 25 characters)`. This was the v1.2.1 defect: two long block names made the upload drop `templates/product.json` and `sections/main-product.liquid`, silently, with no error shown. Check every block name before delivery. Count spaces too. "Personalization options for gifts" is 33 characters.
2. **Section `name` is 25 characters max.** The same 25-character limit applies to the section name in the schema.
3. **Setting `id`s are unique inside each settings array.** A duplicate `id` inside one settings array breaks the schema. Check the section settings, each block type's settings, and `settings_schema.json` separately. A setting `id` may repeat across different scopes (a section setting and a block setting can both be `text`); duplicates only count inside one array.
4. **`img_url` is deprecated. Use `image_url`.** The same applies to `article_img_url`, `collection_img_url`, and `product_img_url`. Theme-check's fix message for all four is: replace the filter with `image_url`. `asset_img_url`, `file_img_url`, and `payment_type_img_url` are not deprecated and are fine to use.
5. **Prices go through the money filter. Never hardcode a currency.** Money comes from `shop.money_format`. A formatted price written literally in code, such as `$49.99`, is an error. A whole-dollar amount in copy, such as `$75`, is a warning, because it is a hardcoded amount that is wrong for stores in another currency.
6. **`structured_data` outputs bare JSON. Always wrap it:** `<script type="application/ld+json">{{ product | structured_data }}</script>` (same for `article`). Bare output renders as visible text on the page.
7. **No placeholder copy visible to shoppers.** These exact phrases must not appear in theme code: "Sample product", "$00.00", "replace with a real customer quote", "Demo text", `placeholder_svg_tag`, "Sample review", "Sample customer".
8. **A section file must exist before a template that references it saves.** Shopify refuses the template save otherwise. Create the section first, then the template.
9. **Layout files must contain `{{ content_for_header }}` and `{{ content_for_layout }}`.** A layout file missing either one fails to save.
10. **Do not chain `default` onto a whole tag.** For example, `alt: x | default: y, class: z` applies `default` to the whole tag and drops the class. Assign the fallback to a variable first.
11. **Every block type a template uses must be defined in that section's schema.** A template that uses a block type the section does not define fails.
12. **Validation is not a visual pass.** A clean run means the code is sound. The theme still needs a preview on a real store and a human eye before it is listed or published.
13. **Do not name a setting id `size`, `first`, or `last`.** Those names are built-in Liquid filters, and settings are read with the same dot notation: `section.settings.size`, `block.settings.size`, `settings.size`. Shopify's own Liquid docs (the metafield object page) document what happens: with dot notation, Liquid returns the value when the key exists, but applies the filter when it is missing, so a missing `size` silently returns a count instead of empty. The setting saves fine, so nothing refuses it; the failure is silent wrong data. The validator flags these three ids as warnings. Rename the id, or read it with bracket notation, for example `section.settings['size']`. Note that `title` is NOT one of these: Shopify's section-schema docs list `heading`, `title`, and `text` as the ids the theme editor uses, in that order, for a block's title in the sidebar. A setting id of `title` is legitimate and does what the merchant expects.

## What theme-check Catches, and What It Misses

Shopify's own validator is theme-check 3.24.0. This skill's validator covers the same ground and adds the checks theme-check misses. The gap is real and proven: the v1.2.1 theme passed theme-check and still lost two files on upload because of long block names.

| Check | theme-check 3.24.0 | This validator |
| --- | --- | --- |
| Section name over 25 characters | ValidSchemaName, error | Error |
| Block name over 25 characters | Missed (ValidSchemaName checks only the section name) | Error |
| Duplicate setting ids in settings_schema.json | UniqueSettingsId, error | Error |
| Duplicate setting ids inside a section or block settings array | Missed (UniqueSettingsId targets settings_schema.json only) | Error |
| img_url, article_img_url, collection_img_url, product_img_url | DeprecatedFilter, error | Error |
| Template references a missing section | MissingTemplate, error | Error |
| Template uses a block type the section does not define | JsonMissingBlock, error | Error |
| Layout missing content_for_header or content_for_layout | RequiredLayoutThemeObject, error | Error |
| Bare structured_data output | Missed | Error |
| Placeholder copy | Missed | Error |
| Hardcoded formatted price, such as $49.99 | Missed | Error |
| Whole-dollar amount in copy, such as $75 | Missed | Warning |
| `default` chained onto a whole tag | Missed | Warning |
| Setting id `size`, `first`, or `last` | Missed | Warning |
| JavaScript syntax | No JS syntax check in theme-check | Checked with node --check when node is present, skipped with a note when it is not |
| Fixing duplicate setting ids | theme-check has no fixer | `validate.py --fix` renames the duplicate and rebinds the Liquid references that read it, writing new files under `fixed/` and never overwriting the input |

## How to Run validate.py

`validate.py` is in the premium pack. It needs Python 3 only, no dependencies.

```bash
python3 validate.py sections/hero-banner.liquid
python3 validate.py /path/to/theme
python3 validate.py sections/hero-banner.liquid templates/index.json
python3 validate.py --fix sections/hero-banner.liquid
python3 validate.py --fix /path/to/theme
```

Pass one file, several files, or a whole theme folder. A folder is scanned recursively for `.liquid` and `.json` files. Cross-file checks (a template referencing a section, a template using a block type) run when the related files are among the checked files. When you pass single files and a referenced section is not among them, the check is skipped with a note, not reported as an error.

--fix repairs duplicate setting ids. For each settings array with a duplicate, it keeps the first occurrence and renames the later ones to `<id>_2`, `<id>_3`, and so on, then rebinds the Liquid references that read them: the first reference to `section.settings.<id>` still reads the first setting, the second reference is rewritten to `section.settings.<id>_2`, and so on. The same rule applies to `block.settings.<id>` references, except when the id is duplicated in more than one block type, in which case the references are left alone and the report says to update them by hand. Bracket-notation references (`section.settings['<id>']`) are not rewritten; the report flags them for hand work. Fixed files are written under `fixed/` in the current directory, mirroring the theme structure, and the original files are never overwritten. Review the fixed files, then move them over the originals yourself. Run `validate.py` again afterwards for the full report.

Output format, one block per file:

```
sections/hero-banner.liquid:
  OK: no errors, no warnings
```

```
sections/hero-banner.liquid:
ERROR: block 'badge' name 'Personalization options for gifts' is 33 chars (>25)
  warning: literal $ amount on line 12; use the money filter
  FAILED: 1 error(s), 1 warning(s)
```

Exit code 1 means the theme has errors and should not be uploaded yet. Warnings never fail the run; they are things to check by hand.

What it checks, in order: JSON parses; section and block names are 25 characters or fewer; setting ids are unique inside each settings array; setting ids named `size`, `first`, or `last` (warning: they collide with Liquid's built-in filters); deprecated image filters; bare structured_data; placeholder copy; hardcoded prices; the default filter-chain trap; layout content_for objects; JavaScript syntax when node is available; template section references exist; template block types are defined in the section schema; template block_order entries exist; a section file with no schema block (warning only, since a section rendered by an API, such as predictive search, legitimately has none).

## Workflow

1. Run `validate.py` on the theme folder.
2. Fix every error. Warnings are notes to check by hand.
3. Push to an unpublished theme first (see the premium checklist), preview it on a real store, and check the routes.
4. Publish only after the preview passes a human eye.

## Worked Example: A Hero Banner

Build a clean hero section. This is `examples/hero-banner.liquid`, and the file in the repo is the exact file the validator passes.

```liquid
<div class="hero" style="--hero-bg: {{ section.settings.bg }};">
  {%- if section.settings.image != blank -%}
    <img src="{{ section.settings.image | image_url: width: 2000 }}" alt="{{ section.settings.heading }}" width="2000" height="1000" loading="eager">
  {%- endif -%}
  <div class="hero__text">
    <h1>{{ section.settings.heading }}</h1>
    {%- if section.settings.subheading != blank -%}
      <p>{{ section.settings.subheading }}</p>
    {%- endif -%}
    {%- if section.settings.button_label != blank and section.settings.button_link != blank -%}
      <a class="button" href="{{ section.settings.button_link }}">{{ section.settings.button_label }}</a>
    {%- endif -%}
</div>
  {%- for block in section.blocks -%}
    <div class="hero__badge" {{ block.shopify_attributes }}>
      <p>{{ block.settings.text }}</p>
    </div>
  {%- endfor -%}
</div>
{% schema %}
{
  "name": "Hero banner",
  "max_blocks": 3,
  "settings": [
    { "type": "image_picker", "id": "image", "label": "Image" },
    { "type": "text", "id": "heading", "label": "Heading", "default": "New collection" },
    { "type": "text", "id": "subheading", "label": "Subheading" },
    { "type": "text", "id": "button_label", "label": "Button label", "default": "Shop now" },
    { "type": "url", "id": "button_link", "label": "Button link" },
    { "type": "color", "id": "bg", "label": "Background", "default": "#FFFFFF" }
  ],
  "blocks": [
    {
      "type": "badge",
      "name": "Badge",
      "settings": [
        { "type": "text", "id": "text", "label": "Badge text", "default": "New in" }
      ]
    }
  ],
  "presets": [
    { "name": "Hero banner", "blocks": [ { "type": "badge" } ] }
  ]
}
{% endschema %}
```

Why this file passes:

- Section name "Hero banner" is 11 characters. Block name "Badge" is 5 characters. Both under 25.
- Setting ids are unique inside each array: `image`, `heading`, `subheading`, `button_label`, `button_link`, `bg` at section level, and `text` inside the badge block. The block setting `text` repeats the idea of the section settings but lives in a different scope, so it is not a duplicate.
- The image uses `image_url`, not the deprecated `img_url`.
- No price is written literally. No placeholder copy. No `structured_data`.

## Troubleshooting

| Symptom | Cause | Fix |
| --- | --- | --- |
| Upload drops a template, such as templates/product.json, with no error shown | A block name over 25 characters in a section the template uses | Shorten every block name to 25 characters or fewer, then re-upload |
| `Invalid block '<type>': name is too long (max 25 characters)` on save | A block name over 25 characters | Shorten the block name in the section schema |
| `Schema name '<name>' is too long (max 25 characters)` | A section name over 25 characters | Shorten the section name in the schema |
| Template save refused | The template references a section file that does not exist yet | Create the section file first, then save the template |
| Template uses a block type the section does not define | The block type is missing from the section schema, or the type is misspelled in the template | Add the block to the section schema, or fix the type in the template |
| Deprecated filter reported | `img_url`, `article_img_url`, `collection_img_url`, or `product_img_url` in use | Replace the filter with `image_url` |
| Structured data renders as visible text on the page | Bare `structured_data` output | Wrap it in `<script type="application/ld+json">` |
| Placeholder copy on the storefront | One of the seven placeholder phrases is still in the theme | Replace it with real copy before delivery |
| `section.settings.size` returns a number, not the setting value | A setting id of `size`, `first`, or `last` | Rename the id, or read it with bracket notation, for example `section.settings['size']` |
| theme-check passes but Shopify still drops files | theme-check does not check block names | Run this validator; it checks block names |
| A file has duplicate setting ids and you want them fixed | Two settings in one array share an id | Run `validate.py --fix` on the file, review the new file under `fixed/`, then move it over the original |

## Gotchas

- Setting ids may repeat across different scopes. A section setting and a block setting can both be `text`. Duplicates only count inside one settings array.
- Renaming a setting id orphans its saved value in templates: template JSON stores settings by id, so after a rename the old value no longer matches and the setting needs to be set again in the editor. `--fix` renames the duplicate and says so in its report.
- A block name counts characters, not words. Count spaces too.
- Do not "fix" a long name by renaming it in the template only. The name lives in the section schema.
- theme-check passing does not mean Shopify accepts the theme. The v1.2.1 theme passed theme-check and still lost two files on upload because of long block names. Run this validator before every upload.
- The validator checks code, not looks. A clean run is not a visual pass.
