# svg-bundle-pipeline

Generate, vectorize, and package commercial-grade, cut-ready SVG designs and bundles for Cricut, Silhouette, Etsy, laser engraving, and sublimation.

## The problem

An AI agent writing SVGs from memory gets them wrong in the same ways, every time:

1. It leaves paths open, and the shape does not cut.
2. It leaves stroke attributes in, and Design Space cuts a thin line instead of a shape.
3. It leaves dust and specks in, and dots appear on the mat that are not in the design.
4. It forgets the even-odd fill rule, and the holes in a stencil do not cut out.
5. It writes files so dense with nodes that Design Space lags or crashes.

This skill teaches the agent the pipeline standards and the five cut-safety rules that keep an SVG cut-ready. They were learned from real commercial SVG work, where each rule is a failure that already happened once.

## What is in the free skill

- `svg-bundle-pipeline/SKILL.md`: the pipeline standards and cut-safety rules, the two output formats, the generate, vectorize, and repair workflows, a worked star example, and a troubleshooting table.
- `svg-bundle-pipeline/references/svg-master-pipeline.md`: the full master pipeline specification, with the real numbers (1024 viewBox, 870 px normalization, 4x Lanczos, dust limits, exact circle fitting, corner detection).
- `svg-bundle-pipeline/INSTALL.md`: install steps for Claude Code, Codex CLI, Cursor, and Claude.ai.
- `svg-bundle-pipeline/examples/star.svg`: the worked example from SKILL.md, a 5-point star in valid cut-ready SVG.
- `svg-bundle-pipeline/examples/cut-ready-checklist.md`: the pre-delivery checklist.

## Install

```bash
mkdir -p ~/.claude/skills
cp -r svg-bundle-pipeline ~/.claude/skills/
```

See `svg-bundle-pipeline/INSTALL.md` for Codex, Cursor, and Claude.ai.

## Premium pack

The premium pack adds four things the free skill does not have:

- `scripts/vectorize_engine.py` and `scripts/vectorize.py`: the vectorization engine and its command-line wrapper. Turn a black-and-white raster silhouette (PNG) into a single-path black stencil SVG. Needs opencv-python, numpy, cairocffi, Pillow, and scipy.
- `validate.py`: a script that checks an SVG file before it goes to a cutting machine. It catches not well-formed XML, a viewBox that is not exactly `0 0 1024 1024`, open paths, missing even-odd fill rule, `<image>`, `<filter>`, `<clipPath>` or `<mask>` elements, stroke attributes, and dust specks under the pipeline dust limit, and warns about normalization, the 8 px border, node count, possible overlaps, and fragile acute angles. It has a `--fix` mode that heals minor issues. Python 3, no dependencies.
- A reference library: four fresh generic shapes in valid cut-ready SVG, a star, a heart, a leaf, and a two-layer flower.
- A Design Space test checklist: the cut-test steps to run on your own machine before you cut a full run or list a design for sale.

Get the premium pack at https://websmartools.com/svg-bundle-pipeline/ for $15, one-time payment, single-user licence.

## Licence

The free skill is MIT licensed. See LICENSE. The premium pack has its own single-user licence and is not open source.
