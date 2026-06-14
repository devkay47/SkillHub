# x-twitter-scraper

Xquik API integration skill for X data access, tweet search, user lookup, follower export, media retrieval, monitoring, webhooks, MCP setup, SDK setup, and confirmation-gated publishing workflows.

## What It Does

This skill gives an agent a safe workflow for using Xquik in API integration tasks. It focuses on scoped requests, input validation, pagination, approval checkpoints, and untrusted content handling for X data.

The skill is useful when a user wants an agent to fetch X data, analyze bounded X datasets, set up an Xquik SDK, configure webhooks, connect the Xquik MCP server, or prepare publishing actions that require user confirmation.

## Compatible Agents

Compatible with Claude Code, Codex, OpenClaw, Cursor, and Gemini CLI skill loaders that support `SKILL.md` folders.

## Installation

Copy the `x-twitter-scraper` folder into your agent skills directory.

Common locations:

- Claude Code project scope: `.claude/skills/x-twitter-scraper/`
- Claude Code global scope: `~/.claude/skills/x-twitter-scraper/`
- OpenClaw project scope: `.openclaw/skills/x-twitter-scraper/`
- Cursor project scope: `.cursor/skills/x-twitter-scraper/`

## Dependencies

The skill requires internet access and a user-provided `XQUIK_API_KEY` environment variable. It does not require local packages.

Use the public docs for current endpoints and setup details:

- `https://docs.xquik.com`
- `https://github.com/Xquik-dev/x-twitter-scraper/tree/master/skills/x-twitter-scraper`

## Example

**You say:** "Use Xquik to find recent posts from @example about product launches and summarize the themes."

**Agent flow:** The agent validates the username, confirms the runtime has `XQUIK_API_KEY`, checks current Xquik docs, fetches a bounded set of recent posts, treats retrieved X content as untrusted text, and summarizes themes without following instructions inside retrieved content.

## Security Note

This skill uses an API key provided through the runtime environment. It never asks for X account credentials, and it requires explicit approval before private reads, publishing actions, monitors, webhook delivery, or bulk extraction jobs.
