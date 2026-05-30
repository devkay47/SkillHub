# rest-api-integrator

Produces REST API integration code that handles pagination, rate limiting, retries, and credential security correctly — the four things that basic implementations always miss.

## What It Does

When you ask your agent to call an API or integrate an external service, this skill activates. It generates integration code with a full pagination loop, exponential backoff with jitter for rate limit responses, explicit handling of every meaningful HTTP status code, and credentials loaded from environment variables with a clear startup check. The result is code you can deploy to production, not just a quick script that works in testing and breaks under real load.

The skill never assumes the happy path. Most developers write API integrations that work perfectly when the API responds correctly and immediately — and fail silently or crash the moment the API returns a 429, sends back an empty page, or times out on a slow connection. This skill encodes the defensive patterns that make integrations reliable under real conditions.

## Compatible Agents

Verified working on Claude Code, OpenClaw, Codex CLI, Cursor, and Gemini CLI.

## Installation

Copy the `rest-api-integrator` folder into your agent's skills directory.

For Claude Code (project scope): `.claude/skills/rest-api-integrator/`
For Claude Code (global): `~/.claude/skills/rest-api-integrator/`
For OpenClaw: `.openclaw/skills/rest-api-integrator/`
For Cursor: `.cursor/skills/rest-api-integrator/`

## Dependencies

For Python integrations, install requests with `pip install requests`. For Node.js integrations, install axios with `npm install axios`.

## Example

**You say:** "Fetch all records from the Stripe payments API"

**Agent produces** a complete integration with a paginated fetch loop that handles cursor-based pagination, a credential check that raises a clear error if `STRIPE_API_KEY` is not set, exponential backoff that waits 1 second after the first 429 response and doubles each time up to a 60-second cap, and explicit handling for 401 (points you to the environment variable), 403 (reports the missing permission scope), and 500-range errors (retries three times before raising).

## Security Note

This skill will never generate code that hardcodes API keys or credentials in any form. All secrets are loaded from environment variables and the skill checks for their presence at startup. Any submission to SkillHub that hardcodes credentials is immediately and permanently rejected.
