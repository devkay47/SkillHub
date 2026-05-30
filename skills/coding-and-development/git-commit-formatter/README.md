# git-commit-formatter

Generates structured, conventional commit messages by analysing your staged Git diff. Install this skill and you will never have to think about commit message format again — the agent reads what changed, selects the correct type and scope, and writes the message for you.

## What It Does

When you have staged changes and ask your agent to help write a commit message, this skill activates automatically. It runs `git diff --staged`, analyses the change, selects the correct Conventional Commits type (`feat`, `fix`, `docs`, `refactor`, etc.), determines the appropriate scope from the affected module or component, and writes a properly formatted subject line and optional body paragraph.

The output follows the [Conventional Commits specification](https://www.conventionalcommits.org/), which is the standard used by the majority of professional open source projects and required by tools like semantic-release and standard-version for automated changelog generation.

## Compatible Agents

Verified working on Claude Code, OpenClaw, Codex CLI, Cursor, and Gemini CLI.

## Installation

Copy the `git-commit-formatter` folder into your agent's skills directory.

For Claude Code (project scope): `.claude/skills/git-commit-formatter/`
For Claude Code (global personal use): `~/.claude/skills/git-commit-formatter/`
For OpenClaw (project scope): `.openclaw/skills/git-commit-formatter/`
For OpenClaw (global personal use): `~/.openclaw/skills/git-commit-formatter/`
For Cursor: `.cursor/skills/git-commit-formatter/`

## Dependencies

None. This skill uses only `git diff --staged`, which is available in any environment with Git installed.

## Example

**You say:** "Help me write a commit message"

**Agent produces:**
