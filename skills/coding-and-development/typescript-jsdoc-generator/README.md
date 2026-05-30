# typescript-jsdoc-generator

Generates complete, accurate JSDoc comments for TypeScript functions, classes, and interfaces. The skill reads your type signatures and infers the correct @param, @returns, @throws, and @example tags automatically.

## What It Does

When you ask your agent to document a TypeScript file or function, this skill activates. It produces JSDoc blocks that are correct for TypeScript's type system, compatible with TypeDoc and JSDoc documentation generators, and written from the user's perspective rather than describing implementation details.

## Compatible Agents

Verified working on Claude Code, OpenClaw, Codex CLI, Cursor, and Gemini CLI.

## Installation

Copy the `typescript-jsdoc-generator` folder into your agent's skills directory.

For Claude Code (project scope): `.claude/skills/typescript-jsdoc-generator/`
For Claude Code (global): `~/.claude/skills/typescript-jsdoc-generator/`
For OpenClaw: `.openclaw/skills/typescript-jsdoc-generator/`
For Cursor: `.cursor/skills/typescript-jsdoc-generator/`

## Dependencies

None. The skill works directly from the TypeScript source text.

## Example

**You say:** "Document this function"

**Agent produces:**
