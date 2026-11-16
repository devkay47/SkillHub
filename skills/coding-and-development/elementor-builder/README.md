# elementor-builder

Build Elementor pages and sections as JSON that imports cleanly the first time.

## The problem

An AI agent writing Elementor JSON from memory gets it wrong in the same five ways, every time:

1. It regenerates style IDs on a transplant, and the import fails with `classes: invalid_value`.
2. It writes a background image as a bare URL, and the background silently drops.
3. It invents a media library attachment ID, and the whole style silently drops.
4. It copies kit-global classes, and the import duplicates them into `DUP_` classes that kill click handlers with no error.
5. It declares responsive settings at the wrong breakpoint, and the tablet layout breaks.

This skill teaches the agent the rules that keep an Elementor JSON import working. They were learned from real client builds, where each rule is a failure that already happened once.

## What is in the free skill

- `elementor-builder/SKILL.md`: the five hard rules, the build rules, the delivery rules, a worked transplant example, and a troubleshooting table.
- `elementor-builder/INSTALL.md`: install steps for Claude Code, Codex CLI, Cursor, and Claude.ai.
- `elementor-builder/examples/section-skeleton.json`: a minimal hero section in valid Elementor JSON, showing the background-image envelope and the style structure. The image ID and URL in it are placeholders. Replace them with real media library values before import. The validator fails the skeleton until you do, on purpose.
- `elementor-builder/examples/transplant-checklist.md`: the step list for copying a subtree between files.

## Install

```bash
mkdir -p ~/.claude/skills
cp -r elementor-builder ~/.claude/skills/
```

See `elementor-builder/INSTALL.md` for Codex, Cursor, and Claude.ai.

## Premium pack

The premium pack adds three things the free skill does not have:

- `validate.py`: a script that checks an Elementor JSON file before import. It catches missing style IDs, broken background envelopes, `DUP_` classes, kit-global risks, and duplicate element IDs, and lists the attachment IDs you need to verify. Python 3, no dependencies.
- A reference library: copy-paste JSON for a hero, a video wrapper with a custom play button, a gallery, a testimonial, and a CTA, plus the canonical envelopes documented for reuse.
- An import QA checklist: the pre-import checks, the import steps, and the post-import class-swap and click-test pass.

Get the premium pack at https://websmartools.com/elementor-builder/ for $15, one-time payment, single-user licence.

## Licence

The free skill is MIT licensed. See LICENSE. The premium pack has its own single-user licence and is not open source.
