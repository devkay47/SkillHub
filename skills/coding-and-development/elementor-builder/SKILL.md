---
name: elementor-builder
description: Build and edit Elementor pages and sections as JSON exports, and transplant Elementor subtrees between files, without breaking imports. Use when creating Elementor JSON, copying a section from one Elementor file to another, adding a background image to a section, or fixing a failed Elementor import. Covers style ID rules, the background-image envelope, kit globals, and the DUP_ class fix.
---

# Elementor Builder

Build Elementor pages and sections as JSON files that import cleanly the first time. Most Elementor JSON that an AI agent writes from memory fails on import, or imports and then silently drops styles, backgrounds, or click handlers. The rules below are the difference. They were learned from real client builds, not from the Elementor docs.

## When to use this skill

Use it when you are asked to:

- Build a new Elementor page or section and deliver it as JSON.
- Copy a section, column, or widget subtree from one Elementor JSON file into another.
- Add a background image, gradient, or overlay to a section or column.
- Fix an Elementor JSON file that failed to import, or that imported with missing styles.
- Fix a page where a click handler or script stopped working after an import.

Do not use it for:

- Full site exports (.wpress files). Deliver JSON only, for sections and pages.
- Live-site edits through the WordPress admin or the database.
- PHP theme work, plugin code, or anything outside Elementor JSON.

## The file shape

An Elementor JSON export is a JSON object with these top-level keys:

```json
{
  "version": "0.4",
  "title": "Page Title",
  "type": "page",
  "page_settings": {},
  "content": []
}
```

`content` is a list of elements. Every element has this shape:

```json
{
  "id": "a1b2c3d4",
  "elType": "e-flexbox",
  "isInner": false,
  "settings": {
    "classes": {
      "$$type": "classes",
      "value": ["g-9ddc351", "e-a1b2c3d4-86db023"]
    }
  },
  "elements": [],
  "styles": {
    "e-a1b2c3d4-86db023": {
      "id": "e-a1b2c3d4-86db023",
      "label": "e-section",
      "type": "class",
      "variants": [
        {
          "meta": {"breakpoint": "desktop", "state": null},
          "props": {}
        }
      ]
    }
  },
  "version": "0.4"
}
```

Three things to notice:

1. `settings.classes` is a typed list. Its entries are class names. Entries starting with `g-` are kit globals. Entries starting with `e-` are local style IDs.
2. `styles` is a dict keyed by style ID. Each style has an `id`, a `label`, a `type` of `class`, and a list of `variants`. Each variant has a `meta` (breakpoint and state) and `props` (the actual declarations, each a typed value with a `$$type`).
3. The local style ID is derived from the element ID: `e-<element-id>-<suffix>`. The suffix is a short hex string. The style lives in the same element's `styles` dict that references it in `classes`.

Widget elements add two keys: `widgetType` (for example `heading`, `text-editor`, `image`, `button`, `video`) and widget-specific settings such as `title` and `tag` for a heading.

## The five hard rules

These are not suggestions. Breaking any one of them produces a file that fails to import, or imports broken.

### Rule 1. Never regenerate style IDs or rename style labels on a transplant

When you copy a subtree from one Elementor JSON file into another, keep every style ID and every style label exactly as it was in the source file. Copy them verbatim.

Regenerating style IDs, or renaming labels to something tidier, makes import validation fail with `classes: invalid_value`, even when you remap the classes lists consistently. Elementor remaps style IDs on import itself. Your job is to hand it the originals untouched.

What is safe: regenerating element IDs is safe but unnecessary. You may leave them as they are.

What is also safe: duplicate style IDs across copied subtrees in one file import fine. Do not try to deduplicate them.

### Rule 2. Background images use one exact envelope

A background image is never a bare URL. It is a `background` prop with this exact nested shape:

```json
{
  "$$type": "background",
  "value": {
    "background-overlay": {
      "$$type": "background-overlay",
      "value": [
        {
          "$$type": "background-image-overlay",
          "value": {
            "image": {
              "$$type": "image",
              "value": {
                "src": {
                  "$$type": "image-src",
                  "value": {
                    "id": {"$$type": "image-attachment-id", "value": 12345},
                    "url": {"$$type": "url", "value": "https://example.com/your-image.jpg"}
                  }
                },
                "size": {"$$type": "string", "value": "full"}
              }
            },
            "repeat": "no-repeat",
            "size": "cover",
            "position": "center center"
          }
        }
      ]
    }
  }
}
```

Every level matters. A missing `$$type`, a missing `background-overlay` wrapper, or an `image-src` without the `image-attachment-id` makes the background silently drop on import.

The `id` in `image-attachment-id` must be the real media library ID of the image on the target site. Never invent one. If you do not know it, ask for it, or use a placeholder and mark the file clearly as needing real IDs before import. A background gradient overlay uses the same outer envelope with `$$type` set to `background-gradient-overlay` and gradient properties inside, and its `angle` is optional. A background that is only a solid color has no overlay at all, just a `color` value inside the background value.

### Rule 3. An attachment ID that does not exist locally drops the whole style

Style validation runs against the media library of the site where the file is validated or imported. If a background image references an attachment ID that does not exist in that library, validation fails and the whole style is silently dropped. The section imports, but with no styling at all, and there is no error message.

So: verify every attachment ID against the target site's media library before you hand a file over. If you are building against a local test site, use the local media IDs in a QA-only variant and swap in the live IDs for the delivered file. Never ship a file whose image IDs you have not checked.

### Rule 4. Kit globals are not validated on import, so do not fix local differences

Classes starting with `g-` are kit globals. They are not validated on import. An unknown global passes through and simply has no CSS on a site whose kit does not define it.

That means a transplant can look wrong on a local test site, for example a play icon rendering at the wrong size, while rendering correctly on the live site where the kit defines the global. Do not fix that local difference by editing the transplant. Leave the `g-` reference alone. The live site is the authority, not your local preview.

### Rule 5. After import, swap DUP_ classes back to the plain class

Importing a template whose elements reference kit globals makes Elementor create duplicate globals labelled `DUP_<name>` instead of reusing the kit's existing ones. The imported elements then carry classes like `DUP_video-wrapper` and `DUP_custom-play-btn`, and any code selecting the plain names silently stops working. Click-to-play goes dead, with no errors anywhere.

Fix it after import, in the editor, on each affected element: remove the `DUP_` class and add the existing plain class. Never rename or delete kit globals. Never edit the script to match the `DUP_` names. Expect this on every import that transplants `g-` references, and budget a class-swap plus click-test pass per page.

## Build rules

These keep a new build clean enough that the hard rules rarely bite.

### Use explicit values, not kit globals, for a clean round trip

When a page must survive an export and re-import with identical rendering, write every color as an explicit hex value and every font as an explicit family and size in the local styles. Do not rely on kit globals for the values that define the design. A file built this way round-trips pixel-identical. A file that leans on kit globals depends on the target kit being identical, which it usually is not.

### Declare responsive settings down the cascade

Elementor settings cascade from desktop to tablet to mobile. Declare each behavior at the breakpoint that owns it.

- For any row that stacks on tablet, set direction to column, gap to 20, and children width to 100 percent at the tablet breakpoint. Never rely on tablet inheriting a mobile setting.
- Give tablet its own explicit padding instead of letting it inherit. Check the tablet breakpoint on its own terms, not as an afterthought of mobile.

Note: exported files usually carry desktop and mobile variants only. Tablet behavior is often set in the kit classes or in the editor after import, so do not expect to see tablet variants in the JSON, and the validator does not check them.

### Use the design system, not local styles

Take backgrounds, gradients, and overlays from the project's design system classes. Never hand-build a local overlay to fake one. Do not add local padding or flex variants that duplicate what a class already does. If the class handles it, leave the local setting empty.

### Keep spacing and copy constants

Learn the project's spacing and copy constants, such as hero padding, section padding, and eyebrow wording, from the project's existing files and reuse them exactly. Do not invent your own constants when the project already has them. A constant you guess is a correction someone has to make. Build the constant set for a new project from its first real file and grow it with each round. On a brand-new project with no existing files, state the constants you chose (padding, colors, type scale) in the delivery note so the owner can correct them in one pass, instead of hiding them in the file.

## Delivery

- Deliver JSON exports for sections and pages only. Never a full .wpress file.
- Give every file a distinct versioned name, for example `homepage-v1.json`, `homepage-v2.json`. Never overwrite a delivered file with a new version under the same name.
- Before you hand a file over, run the checks in the troubleshooting table below. A file that has not been checked has not been delivered.
- If you do not know a media library ID, a kit global name, or a project constant, say so in the delivery note. Do not invent one to make the file look complete.

## Worked example: transplant a section

The task: copy a hero section from `source-page.json` into `target-page.json`.

1. Open both files. Find the hero section element in the source, the top-level `e-flexbox` element and everything nested under it.
2. Copy the whole subtree verbatim into the target's `content` list. Do not touch any style ID or style label inside it. Do not rename anything.
3. Element IDs may stay as they are. If two elements in the target file now share an ID, regenerate the copied element's ID and update any style ID derived from it in the same subtree, keeping the style label unchanged.
4. Check every `background` prop in the copied subtree against the envelope in Rule 2. Check every `image-attachment-id` against the target site's media library. Replace any ID that does not exist on the target.
5. Scan the copied classes for `g-` entries. Note each one: it needs the kit on the target site to define it, and the import may create a `DUP_` copy.
6. Import the target file. Then run the Rule 5 fix: on each imported element, remove any `DUP_` class and add the plain class. Test every click handler on the page.
7. Check the page at desktop, tablet, and mobile widths before calling it done.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Import fails with `classes: invalid_value` | A class is listed in `settings.classes` but has no matching style in the same element's `styles` dict, or a style ID was regenerated or a label renamed during a transplant | Remove the dangling class, or restore the missing style from the source file. If it was a transplant, restore the original style IDs and labels verbatim |
| Section imports with no styling at all | A background image references an attachment ID that does not exist in the target media library | Replace the ID with a real media library ID from the target site, or remove the background |
| Background image missing after import | The `background` prop does not match the envelope in Rule 2 | Rewrite the prop with the exact nested shape, including `image-attachment-id` and `size: full` |
| Click handler dead after import, no errors | The import created `DUP_` copies of kit-global classes | Remove the `DUP_` class from each affected element and add the plain class, in the editor |
| Page looks wrong locally but right on live | Kit globals in the file are not defined in the local kit | Do nothing to the file. The live kit is the authority |
| Tablet layout breaks while desktop and mobile are fine | Tablet is inheriting instead of declaring its own settings | Set direction, gap, children width, and padding explicitly at the tablet breakpoint |

## Never invent

- Never invent a media library attachment ID. Use a real one or mark the file as needing it.
- Never invent a kit-global class name. Use only classes that exist in the project's kit, or local `e-` styles you define.
- Never invent a style label. Copy the label from the source, or use the element's own naming pattern.
- Never invent spacing, padding, or copy constants. Take them from the project.
