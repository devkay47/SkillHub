# Document Processing Skills

This category contains skills that teach AI agents how to work correctly and professionally with structured document formats — PDFs, Word documents, Excel spreadsheets, and PowerPoint presentations.

## What Belongs Here

A skill belongs in this category if its primary value is handling a specific file format correctly. This means reading from files without losing structure, writing to files with proper formatting, filling forms, merging or splitting documents, extracting tables, generating charts inside spreadsheets, or producing output that meets professional document standards.

If the skill is primarily about analysing data that happens to be in a spreadsheet, it may belong in Data Analysis. If it is about automating a document-heavy workflow end to end, it may belong in Productivity Automation. The question to ask is: is the core technical challenge in this skill about the file format itself? If yes, it belongs here.

## Why This Category Matters

Agents notoriously struggle with document formats. Without a skill, an agent asked to create a Word document may produce output that looks correct in plain text but breaks when opened in Microsoft Word — missing styles, broken headers, corrupted tables. The skills in this category encode the hard-won, format-specific knowledge that prevents these failures. Each skill here represents hours of experimentation distilled into reusable instructions.

## Quality Bar For This Category

Document processing skills must specify exactly which library or tool they use to produce output. A skill that says "create a Word document" without specifying python-docx, pandoc, or another concrete tool cannot be verified. Reviewers will test the skill by running it — if the output file does not open correctly in the target application, the skill will not be verified.

## Skills In This Category

| Skill | Description | Compatible With |
|---|---|---|
| `pdf-text-extractor` | Extracts text and tables from PDFs, handling multi-column layouts and scanned documents | Claude Code, Codex, OpenClaw, Cursor |
| `docx-professional-writer` | Creates properly structured Word documents with styles, headings, and tables using python-docx | Claude Code, Codex, OpenClaw, Cursor |

## Installing a Skill From This Category

Copy the skill folder into your agent's skills directory. For Claude Code and OpenClaw, use `.claude/skills/` or `.openclaw/skills/` inside your project, or `~/.claude/skills/` for global personal use.

## Submitting a Skill To This Category

Read the main [CONTRIBUTING.md](../../CONTRIBUTING.md) before submitting. Document processing skills must include the name of every library dependency in the `metadata.json` and must note which versions have been tested. A skill that works on python-docx 0.8 but breaks on 1.1 is a problem the reviewer needs to know about upfront.
