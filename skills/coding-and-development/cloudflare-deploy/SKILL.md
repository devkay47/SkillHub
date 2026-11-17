---
name: cloudflare-deploy
description: A Claude Code skill for setting up and diagnosing Cloudflare Pages, Workers, DNS, Email Routing, and D1. Use when the user asks to set up a site on Cloudflare, deploy or redeploy a Pages site or a Worker, add a binding, secret, or cron trigger, or diagnose a deploy that serves errors, stale content, or nothing at all.
---

# Cloudflare Deploy

Set up and deploy a site on Cloudflare without the failures that cost a day each. This skill is the deploy procedure for a static site on Cloudflare Pages plus an API Worker, with DNS, Email Routing, and D1. It is written for Claude Code. Every rule below is a failure that already happened once on a real project.

## When to Use

- The user asks to set up a new site on Cloudflare (domain, DNS, Pages, Worker, Email Routing, D1).
- The user asks to deploy or redeploy a Pages site or a Worker.
- The user asks to add a binding (D1, KV, R2, rate limit, queue), a secret, or a cron trigger to a Worker.
- A deploy went out and the site serves an error, stale content, or nothing at all.
- The user asks to check a deployment before it goes live.

## Reference Files

Load the relevant reference before starting work:

- `references/setup-checklists.md`: the setup checklists for DNS, Pages, Workers, Email Routing, and D1. Read the section for the part you are setting up.
- `references/error-diagnosis.md`: the error-diagnosis guide. When a deploy is broken, start here, not by guessing.
- `examples/wrangler-example.toml`: a complete, good example Worker config. Copy its shape, replace every example value.
- `examples/deploy-config-example.json`: an example deploy config for the premium deploy-check script. It records what the script cannot read from `wrangler.toml`.
- `examples/deploy-checklist.md`: the pre-deploy checklist. Run through it before every deploy.

## The Six Hard Rules

These are not suggestions. Each one is a scar.

### Rule 1: Pages deploys go through Wrangler only

Run `wrangler pages deploy build --project-name=<name> --branch=main`. Never deploy a Pages site through a direct-upload route. A direct upload once produced a deployment that served HTTP 500 on every path.

### Rule 2: Every Worker redeploy must pass the full bindings

A Worker deploy replaces the Worker metadata. Any binding not listed in the deploy is wiped: D1 database, rate limits, KV, R2, queues. Always pass the full bindings list on every redeploy, even if you changed nothing about the bindings. Replacing a Worker is not the same as updating it.

### Rule 3: Check the compatibility date before you deploy

`wrangler.toml` has a `compatibility_date`, and the deploy tool has its own date. When they drift apart, the Worker can behave differently from the config, and Node APIs can be silently unavailable. Check them and align them before you deploy. The mismatch once found was small: `wrangler.toml` said 2026-10-01 while the deploy tool used 2024-11-01. It was not a live defect that time, but it is the kind of silent drift that is.

### Rule 4: Never run two builds or deploys against the same build folder at the same time

Sequence them. Two concurrent builds writing to the same output folder corrupt each other, and two concurrent deploys of the same folder produce inconsistent results. A failed build once looked like a failed deploy because the two ran at the same time.

### Rule 5: One permanent full-scope token, stored once, never rolled

Use one permanent Cloudflare API token with the scopes the project needs, stored once in the secure store the deploy tool uses. Never create a new token for each session and never roll the token mid-project: rolling it breaks every stored credential that references it.

### Rule 6: Bump the app version query string on every app.js change, and verify live after every deploy

Browsers cache JavaScript by URL. If you change `app.js` without changing its version query string, browsers keep serving the old copy and your fix looks broken. After every deploy, verify live: check the homepage and one tool page return HTTP 200, and that the new content is in the served HTML. A successful command exit is not a successful deploy.

The premium deploy-check script catches this before you deploy: `python3 deploy-check.py wrangler.toml --dist build` scans the built HTML for script and stylesheet references and warns on any local `.js` or `.css` reference without a version query string. A reference counts as versioned when it has a query string containing `v=` or `ver=` and a non-empty value. A bare `?` does not count, and neither does `?x=1`.

## Core Workflow

### 1. First setup (new project)

Follow the order in `references/setup-checklists.md`. The order is: domain and DNS, then the Pages project, then the Worker, then routes, then secrets, then cron triggers. Do not skip the DNS propagation wait: a Worker deployed before DNS has propagated serves nothing.

### 2. Pages deploy

1. Build the site into the build folder.
2. Check nothing else is building or deploying against the same folder (Rule 4).
3. Run the cache-busting check on the build folder (`python3 deploy-check.py wrangler.toml --dist build` in the premium pack). Fix any unversioned reference it warns about before you deploy.
4. Run `wrangler pages deploy build --project-name=<name> --branch=main`.
5. Verify live: homepage 200, one tool page 200, the new content is in the served HTML.
6. If every path serves 500, redeploy the same build through Wrangler (Rule 1) before investigating anything else.

### 3. Worker redeploy

1. List every binding the Worker uses (D1, rate limits, KV, R2, queues). Write the list down.
2. Check the compatibility date against the deploy tool (Rule 3).
3. Deploy with the full bindings list (Rule 2).
4. Verify live: the Worker routes respond, the bindings are still listed in the dashboard, the cron triggers are still listed.
5. If a binding is missing after the deploy, redeploy with the full list. Do not patch bindings by hand in the dashboard.

### 4. Adding a binding, secret, or cron trigger

1. Add it to the config file (wrangler.toml) first, so the next deploy carries it.
2. Set secrets with `wrangler secret put` or in the dashboard. Secrets are never written in a config file and never pasted into chat.
3. Redeploy with the full bindings list (Rule 2). Adding one binding is not a reason to drop the others.
4. Verify the new binding or trigger works live, and that the old ones still do.

### 5. Diagnosing a broken deploy

Do not guess. Open `references/error-diagnosis.md` and work the table in order: symptom, likely cause, check, fix. The most common causes, in order, are: a Pages direct upload (Rule 1), a Worker redeploy that dropped a binding (Rule 2), a compatibility date drift (Rule 3), and a stale cached `app.js` (Rule 6).

## Plan limits worth knowing

These are the Cloudflare plan limits as documented in October 2026. Confirm them on Cloudflare's pricing page before quoting them to a user.

- Workers Free: 100,000 requests per day, 5 cron triggers, 50 subrequests per invocation, D1 100,000 rows read and 50,000 rows written per day, D1 10 GB storage, R2 10 GB storage.
- Workers Paid ($5 per month): 10 million requests per month included, Browser Rendering. The Paid plan includes the Browser Rendering API.
- Cloudflare blocks outbound port 25 from Workers. A port checker will report SMTP port 25 as closed on a Worker. That is not a bug.
- Bot Fight Mode blocks automated capture services on the main domain. If an external screenshot or checker cannot reach the site, test it on the `*.pages.dev` URL instead. Check the settings before calling it a deploy failure.

## Gotchas

- Never write a secret, token, account ID, zone ID, or database ID into a config file, a skill file, or chat. Use `wrangler secret put` or in the dashboard. IDs are placeholders in examples until the real values are put in the real config.
- Never deploy a Worker without a route and then wonder why the domain returns nothing. Without a route, the Worker is deployed but unreachable.
- Never trust a deploy you have not checked live. A successful command exit is not a successful deploy.
- Never run a build and a deploy at the same time (Rule 4). The failure looks like a deploy bug and wastes a day.
- Never roll the Cloudflare token mid-project (Rule 5). If a token stops working, check the scopes first, then the stored credential, then create a replacement.
- A compatibility date in the future is invalid. Cloudflare rejects it. A date older than 2021 is stale and may behave unexpectedly.
- Cron triggers are limited to 5 on the Free plan. A sixth trigger silently does not run.
