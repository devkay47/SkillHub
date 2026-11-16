---
name: svg-bundle-pipeline
description: Commercial cut-ready SVG generation, vectorization, and bundle packaging pipeline. Use when the user asks to generate, vectorize, trace, or package SVG cutting files, stencil silhouettes, layered color vectors, or commercial SVG bundles (e.g., for Cricut, Silhouette, Etsy, laser engraving, or sublimation).
---

# SVG Bundle Pipeline

Generate, vectorize, validate, and package commercial-grade, cut-ready SVG designs and bundles in both single-path compound stencil (`fill-rule="evenodd"`) and multi-layer flat-color vector formats.

## When to Use

- The user asks to generate or vectorize designs into `.svg` files (single designs, pilot prototypes, or multi-design batches).
- The user uploads or generates raster reference artwork and asks to convert it into cut-ready SVGs.
- The user requests an SVG bundle packaged with transparent PNG previews, a visual contact sheet (`00-Bundle-Preview-Catalog.png`), and technical documentation.
- The user mentions "Vectorize Batch X", "SVG bundle", "cut-ready SVG", "layered color SVG", or "stencil cut SVG".
- The user asks to fix a broken SVG, check whether an SVG will cut cleanly, or simplify paths, remove specks, enforce closed paths, or convert raster art to vector.

## Reference Files and Helper Scripts

Before executing the vectorization or packaging workflow, read the relevant auxiliary files by path:

- `references/svg-master-pipeline.md` - Complete master pipeline specification and formatting standards. Read it before building a design.
- `scripts/vectorize_engine.py` (premium pack) - The reusable vectorization engine: 4x Lanczos supersampling, adaptive least-squares cubic Bezier fitting, exact circle recognition, Cairo rendering, and ZIP packaging. It produces only the single-path black stencil SVG. Run it through the command-line wrapper, `scripts/vectorize.py` (premium pack).
- `validate.py` (premium pack) - Pre-import validator. Run it on every SVG before delivery; it checks the pipeline standards and the cut-safety rules below.

## Core Workflow

### 1. Pilot-First Execution (For New Series)

When starting a new design series or converting a new style sheet:

1. Generate or vectorize **Design 1** first as a pilot sample (`01-<Name>-Prototype.zip` or `<Number>-<Name>-Prototype.zip`).
2. Render a high-resolution preview image and inspect it visually to verify clean geometry, zero clipped edges, and smooth curves.
3. Pause and ask the user to approve the pilot before scaling to full batches of 5 or 30 designs.

### 2. Two-Turn Raster-to-SVG Workflow (When Generating New Raster Silhouettes)

When the raster silhouettes come from an image generator or another tool:

1. **Step 1:** Obtain individual high-contrast black-and-white woodcut/linocut stencil silhouettes on a pure solid white background (`#FFFFFF`) with a clear outer margin and zero text, watermarks, or grey shading. Get them as PNG files before doing anything else.
2. **Step 2:** Run the vectorization engine on the PNG files (`scripts/vectorize.py input.png`) to produce true cubic Bezier `.svg` files. Do not hand-trace what the engine can vectorize.

### 3. Direct Vector Construction and Image Tracing (For Uploaded Reference Sheets)

When the user uploads an existing reference image or multi-item sheet:

1. Crop each design and overlay a coordinate grid (`20 px` intervals) to map exact proportions and sample hex colors.
2. Produce **both** vector versions when color artwork is provided (unless the user specifies only one):
   - **Multi-Layer Color SVG (`*-Color.svg`):** Separate `<path id="...">` elements grouped by color inside a `<g>` container (`viewBox="0 0 1024 1024"`) for layered cardstock or multi-color HTV cutting. Order the layers back-to-front: base and background shapes first, then the main body, then middle details, then foreground features, then the smallest top accents last.
   - **Single-Path Black Stencil Cut SVG (`*-Stencil.svg`):** Compound `<path fill-rule="evenodd" fill="#111827" d="..." />` (`viewBox="0 0 1024 1024"`) with crisp negative-space separation moats between overlapping shapes. Use `14 px` to `24 px` moats (`#FFFFFF`) or bold outlines (`#000000`) so every feature remains distinct and weedable.

### 4. Quality and Normalization Standards

Apply these standards to every generated `.svg` file. The full specification is in `references/svg-master-pipeline.md`.

- **ViewBox and Normalization:** Use `viewBox="0 0 1024 1024" width="100%" height="100%"`. Center every design at `(512, 512)` and normalize its longest dimension to `870 px` (`85%` of the canvas) so a clean border remains on all sides (at least a `77 px` safety margin).
- **4x Supersampling and Dust Filtering:** Upscale raster inputs `4x` (`4096x4096`) with `cv2.INTER_LANCZOS4` and Gaussian smoothing (`(7, 7), 1.6`) before thresholding at `180` (`cv2.THRESH_BINARY_INV`). Discard micro-specks (`area < 350` in 4x space, or `< 650` when the file has over 120 subpaths; `perim < 10.0` in 1x space) so vinyl cutters do not snag.
- **Exact Cubic Bezier Circles and Smooth Curves:** Convert small circular cutouts (`perim_norm < 55.0` and `circularity > 0.82`) into exact 4-segment cubic Bezier circles (`k = 0.55228475 * r`). Fit all other contours with adaptive least-squares cubic Bezier curves (`C`) and straight lines (`L`). Detect sharp corners (`angle > 48 deg`) and fit smooth chains between them (`max_err = 0.9` to `1.0`).
- **Programmatic XML and Visual QA:** Parse every `.svg` with `xml.etree.ElementTree` to verify well-formed XML, and render every `.svg` with `cairocffi` (`FILL_RULE_EVEN_ODD`) to inspect the output image before packaging.
- **Margin Clearing:** Always clear an `8 px` white border on raster inputs before contour extraction so frame edges never become an outer bounding box path.

### 5. Cut-Safety Rules

These five checks sit on top of the pipeline standards. They are what make a file cut cleanly on a real machine, and the validator checks all five.

1. **Node budget.** Flag any file over about 1,000 nodes for HTV, or about 1,500 for permanent vinyl. Design Space slows or stalls above these; simplify the paths (fewer points, smoother curves) until the count is under budget.
2. **Stroke-to-fill.** Flag any remaining `stroke` attribute. Output filled shapes only. A cutter follows the centerline of a stroke, not its visible width, so a stroked shape cuts as a hairline.
3. **Overlaps.** Overlapping paths in the same fill must be welded or subtracted. Two paths that cross cut into each other and leave a ragged edge where they meet.
4. **Acute angles.** Internal corners sharper than 30 degrees on short edges tear vinyl during weeding. Soften the corner or thicken the spike.
5. **ViewBox integrity.** The viewBox must be present and exactly `0 0 1024 1024`. A missing or damaged viewBox imports at a massive or microscopic scale.

### 6. Packaging and Clean Delivery

Package deliverables into a single `.zip` archive in the current working directory:

1. Stage all intermediate images, previews, and folder structures inside `/tmp/` so loose `.svg` or `.png` files do not clutter the workspace.
2. Include at the **root level** of the `.zip`:
   - Standalone `.svg` cutting files (`01-...svg`, etc.)
   - Numbered visual reference sheet (`00-Bundle-Preview-Catalog.png`)
3. Include a structured subfolder inside the `.zip` containing:
   - `SVG/` (or `SVG-Layered-Color/` and `SVG-Stencil-Cut/`)
   - `PNG-Previews/` (`1024x1024` transparent background PNGs)
   - `Preview-Sheet/` (`00-Bundle-Preview-Catalog.png`)
   - `README.md` and `Documentation/` (design notes and per-file pre-flight metrics: sub-path count, exact Bezier circle count, file size in KB, and XML validation status).
4. Run `validate.py` on every SVG before packaging and fix anything it flags as an error. Warnings are notes, not blockers; errors are not shipped.

## Worked Example: A Five-Point Star

Build a cut-ready star, single-path compound stencil format. The star is centered at `(512, 512)`, outer radius `435`, inner radius `174`, longest dimension `870 px`, exactly as the normalization standard requires.

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1024 1024" width="100%" height="100%">
  <path fill-rule="evenodd" fill="#111827" d="M 512.00 98.29 L 619.53 407.67 L 947.00 414.28 L 686.00 612.16 L 780.83 925.71 L 512.00 738.60 L 243.17 925.71 L 338.00 612.16 L 77.00 414.28 L 404.47 407.67 Z"/>
</svg>
```

Why this file passes:

- One closed path, ten straight segments, ending in `Z`. No open subpath.
- `viewBox="0 0 1024 1024"`, design centered at `(512, 512)`, longest dimension `870 px`.
- Single fill `#111827`, `fill-rule="evenodd"` (harmless on a single subpath, required once a second subpath appears).
- 10 commands, far under the 1,000 node HTV budget.
- No stroke, no `<image>`, no `<filter>`, no `<clipPath>`. No specks.
- The star points are sharp, but each point sits on long edges (over 300 units), so they are not fragile spikes and the acute-angle check stays quiet.

A two-layer version of the same idea is in `examples/star.svg` (single color) and in the premium `references/flower-two-layer.svg` (petals in one color, center in another, one `<path>` per color group inside a `<g>`).

## Gotchas

- Never use polygon-only (`L`-command) contour dumps from raw `cv2.approxPolyDP`; always fit smooth cubic Bezier (`C`) curves so cutting blades glide smoothly.
- Never embed `<image>` raster tags, `<filter>`, or `<clipPath>` elements inside cut-ready SVGs.
- Always clear an `8 px` white border on raster inputs before contour extraction so frame edges never become an outer bounding box path.
- Never ship a file the validator flags as an error. If the engine drops a speck, that is the dust filter working, not a missing shape.
- A file with a different viewBox is valid SVG but does not follow this pipeline; normalize it to `0 0 1024 1024` before delivery.

## Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| File imports at a huge or tiny size | viewBox missing, or not `0 0 1024 1024` | Set viewBox to `0 0 1024 1024` and normalize the design to 870 px centered at (512, 512) |
| Holes do not cut out | Compound path without `fill-rule="evenodd"` | Add `fill-rule="evenodd"` to the path |
| Cutter draws a hairline instead of the shape | A `stroke` attribute is still set | Remove the stroke; output filled shapes only |
| Machine snags or skips a tiny dot | A dust speck under the area and perimeter limits | Let the engine drop it, or delete the subpath by hand |
| Design Space lags or stalls on load | Node count over the HTV (1,000) or vinyl (1,500) budget | Simplify the paths; fewer points, smoother curves |
| Two shapes cut into each other | Overlapping paths in the same fill | Weld or subtract the overlap before export |
| Vinyl tears when weeding a point | An acute angle under 30 degrees on a short spike | Soften the corner or thicken the spike |
| A ragged edge where two layers meet | No separation moat in the stencil version | Add a 14 to 24 px white moat or a bold black outline between the layers |
| The outer frame cuts as a rectangle | The 8 px border was not cleared before tracing | Clear the outer 8 px of the raster input and re-run the engine |
| The vectorized shape looks jagged | Polygon-only output, no Bezier fitting | Re-run the engine; never ship a raw `approxPolyDP` dump |
| The engine finds no contours | Input is not a solid black silhouette on white | Check the input is high-contrast black on pure white with a clear margin |

## Delivery Rules

- Deliver a `.zip` only, built as in Packaging above. Never hand over loose intermediate files.
- Run `validate.py` on every SVG and include the per-file pre-flight metrics (sub-path count, exact Bezier circle count, file size in KB, XML validation status) in `Documentation/`.
- Never claim a cut test was run. The validator is a static pre-check; the buyer test-cuts on their own machine (see the Design Space test checklist in the premium pack).
- Never copy a real product design into a new bundle. Reference shapes in this skill (star, heart, leaf, flower) are generic teaching shapes, not catalogue artwork.
- When the user asks for a new series, start with Design 1 as a pilot and wait for approval before batching.
