# pdf-text-extractor

Extracts text, tables, and metadata from PDF files reliably — including multi-column layouts and scanned documents that require OCR.

## What It Does

When you ask your agent to read, parse, or extract content from a PDF, this skill activates. It detects the PDF type automatically (text-based, multi-column, or scanned) and applies the appropriate extraction strategy. Tables are returned as structured data (list of dicts), text preserves page boundaries, and metadata is always included alongside the content.

## Compatible Agents

Verified working on Claude Code, OpenClaw, Codex CLI, Cursor, and Gemini CLI.

## Installation

Copy the `pdf-text-extractor` folder into your agent's skills directory.

For Claude Code (project scope): `.claude/skills/pdf-text-extractor/`
For Claude Code (global): `~/.claude/skills/pdf-text-extractor/`
For OpenClaw: `.openclaw/skills/pdf-text-extractor/`
For Cursor: `.cursor/skills/pdf-text-extractor/`

## Dependencies

Install required Python packages with `pip install pdfplumber pdf2image pytesseract Pillow`. For scanned document support, also install Tesseract OCR on your system — installation instructions are available at tesseract-ocr.github.io.

## Example

**You say:** "Extract the text and tables from this PDF"

**Agent produces:**
