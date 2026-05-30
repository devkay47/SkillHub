# API Integration Skills

This category contains skills that teach AI agents how to work with external APIs and web services reliably — handling authentication, pagination, rate limiting, error recovery, and format-specific quirks that are rarely documented clearly.

## What Belongs Here

A skill belongs in this category if its primary value is helping an agent interact correctly with an external service. This includes OAuth and API key authentication flows, making paginated requests without losing data, handling rate limit errors with exponential backoff, parsing API responses that have inconsistent structure, working with webhooks, and building reliable integrations that handle failure gracefully.

If the skill is primarily about transforming or analysing data that came from an API, it may belong in Data Analysis. If it is about automating an entire workflow that happens to use an API as one step, it may belong in Productivity Automation. The question to ask: is the core technical challenge in this skill the API interaction itself? If yes, it belongs here.

## Why This Category Matters

APIs are where agents fail quietly. An agent that does not handle rate limiting will work perfectly in testing and break in production. An agent that does not handle pagination will silently return only the first page of results and present them as complete. An agent that does not implement retry logic will fail on transient network errors and report the task as impossible. The skills in this category encode the defensive patterns that make API integrations reliable rather than fragile.

## Quality Bar For This Category

API integration skills must document every external service they interact with in the `metadata.json`. They must explicitly address error handling — what happens when the API returns a 429, a 500, or an unexpected response format. Skills that assume the happy path only will not pass review. Additionally, no skill in this category may hardcode API keys or credentials in any file. All secrets must be passed via environment variables, and the skill must document this clearly.

## Skills In This Category

| Skill | Description | Compatible With |
|---|---|---|
| `rest-api-integrator` | Integrates REST APIs with automatic pagination, rate limit handling, and structured error recovery | Claude Code, Codex, OpenClaw, Cursor |

## Installing a Skill From This Category

Copy the skill folder into your agent's skills directory. For Claude Code and OpenClaw, use `.claude/skills/` or `.openclaw/skills/` inside your project, or `~/.claude/skills/` for global personal use.

## Submitting a Skill To This Category

Read the main [CONTRIBUTING.md](../../CONTRIBUTING.md) before submitting. API integration skills undergo stricter security review than other categories because they interact with external services and handle credentials. Any skill found to log, expose, or hardcode credentials will be permanently rejected and the submission flagged.
