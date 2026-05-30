# code-reviewer

Performs a structured code review organised by severity — Critical, Warning, and Suggestion — so you know exactly where to focus your attention first. Works on any language, any file size.

## What It Does

When you ask your agent to review code, this skill activates and runs a four-dimensional analysis covering correctness bugs, security vulnerabilities, performance problems, and style issues. Every finding is assigned to a severity tier with a clear explanation and, for critical issues, a corrected code snippet showing exactly how to fix it.

The structured output is designed to be immediately actionable. You can copy the Critical section directly into a PR review comment, send the full output to a teammate, or work through it top-to-bottom as a fix list.

## Compatible Agents

Verified working on Claude Code, OpenClaw, Codex CLI, Cursor, and Gemini CLI.

## Installation

Copy the `code-reviewer` folder into your agent's skills directory.

For Claude Code (project scope): `.claude/skills/code-reviewer/`
For Claude Code (global): `~/.claude/skills/code-reviewer/`
For OpenClaw: `.openclaw/skills/code-reviewer/` or `~/.openclaw/skills/code-reviewer/`
For Cursor: `.cursor/skills/code-reviewer/`

## Dependencies

None.

## Example Output

### 🔴 Critical
- Line 34: Hardcoded database password in plaintext. Anyone with read access to this file can access the production database. Move to an environment variable immediately.

### 🟡 Warning
- Line 67: Missing await on async function call. This will silently return a Promise object instead of the resolved value, causing downstream type errors.

### 🔵 Suggestion
- Lines 89-124: This function is handling three separate concerns (validation, transformation, persistence). Consider splitting into three focused functions for easier testing.

### ✅ What Works Well
Error handling in the API layer is thorough and consistent. The use of typed interfaces throughout makes the data flow easy to follow.

## Notes

The skill reviews whatever code you point it to — a pasted snippet, a named file, or a staged diff. For very large files, it focuses the Critical and Warning sections on the highest-risk areas rather than producing an exhaustive line-by-line audit.
