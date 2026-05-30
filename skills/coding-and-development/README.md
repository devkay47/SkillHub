# Coding and Development Skills

This category contains skills that make AI coding agents significantly better at the core tasks of software development — writing code, reviewing it, refactoring it, documenting it, and testing it.

## What Belongs Here

A skill belongs in this category if it improves how an agent handles any of the following: writing new code to a specific standard or pattern, reviewing existing code for bugs or security issues, refactoring code while preserving behaviour, generating tests, writing documentation, managing Git workflows, or applying language-specific or framework-specific best practices.

If the skill is primarily about working with a file format (PDF, Excel, Word), it belongs in Document Processing. If it is primarily about connecting to an external service, it belongs in API Integration. If it is about automating a workflow that happens to involve code, it may belong in Productivity Automation. When in doubt, ask yourself: is the core value of this skill about writing better software? If yes, it belongs here.

## Quality Bar For This Category

Coding skills are the highest-demand category in SkillHub and therefore have the highest scrutiny during review. A skill in this category must be specific enough that a developer can read the description and know exactly which tasks it handles and which it does not. "Write better code" is not a skill. "Generate JSDoc comments for TypeScript functions, including parameter types, return values, and usage examples" is a skill.

## Skills In This Category

| Skill | Description | Compatible With |
|---|---|---|
| `git-commit-formatter` | Generates structured, conventional commit messages from staged diffs | Claude Code, Codex, OpenClaw, Cursor |
| `code-reviewer` | Reviews code for bugs, security issues, and style violations with severity ratings | Claude Code, Codex, OpenClaw, Cursor |
| `typescript-jsdoc-generator` | Generates complete JSDoc comments for TypeScript functions and classes | Claude Code, Codex, OpenClaw, Cursor |
| `test-suite-generator` | Generates comprehensive unit and integration tests for a given function or module | Claude Code, Codex, OpenClaw, Cursor |

## Installing a Skill From This Category

Copy the skill folder into your agent's skills directory. For Claude Code and OpenClaw, this is `.claude/skills/` or `.openclaw/skills/` inside your project, or the global `~/.claude/skills/` path for personal use across all projects.

## Submitting a Skill To This Category

Read the main [CONTRIBUTING.md](../../CONTRIBUTING.md) before opening a Pull Request. For coding skills specifically, your submission must include at least one concrete example in the SKILL.md body showing the agent's expected input and output — this is what reviewers use to verify the skill actually works.
