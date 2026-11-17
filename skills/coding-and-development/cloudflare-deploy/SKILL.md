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
- `examples/deploy-config-example.json`: an example deploy config for the premium deploy-check script. It records what the script cannot read from wrangler.toml: the Pages project and branch, the deploy tool's compatibility date, the bindings the deploy must carry, and the secrets the Worker needs.
- `examples/deploy-checklist.md`: the pre-deploy checklist. Run through it before every deploy.

## The Six Hard Rules

These are not suggestions. Each one is a scar.

### Rule 1: Pages deploys go through Wrangler only

Run `wrangler pages deploy build --project-name=<name> --branch=main`. Never deploy a Pages site through a direct-upload route. A direct upload once produced a deployment that served HTTP 500 on every path, including the homepage. The same build redeployed through Wrangler fixed it immediately. If a Pages deploy serves 500 on every path, redeploy the same build through Wrangler before you investigate anything else.

### Rule 2: Every Worker redeploy must pass the full bindings

A Worker deploy replaces the Worker metadata. Any binding not listed in the deploy is wiped: D1 databases, rate limit bindings, KV namespaces, R2 buckets, queues. Cron schedules and secrets survive a redeploy. Bindings do not. Before every Worker redeploy, list every binding the Worker uses and pass all of them. After the deploy, check that the bindings are still there. If a binding is gone after a deploy, redeploy with the full bindings list. Never assume a deploy kept them.

### Rule 3: Check the compatibility date before you deploy

`wrangler.toml` has a `compatibility_date`, and the deploy tool has its own date. When they drift apart, the Worker can behave differently from the config you wrote. A drift was found on a real project: `wrangler.toml` said 2026-10-01 while the deploy tool used 2024-11-01. It was not a live defect that time, but it is a real failure mode. Compare the two dates before every deploy. If they differ, decide which one is right and make them match. Never deploy with a date you have not looked at.

### Rule 4: Never run two builds or deploys against the same build folder at the same time

Sequence them. Two concurrent builds writing to the same output folder corrupt each other, and two concurrent deploys of the same project race. One at a time, in order, with the live check after each one.

### Rule 5: One permanent full-scope token, stored once, never rolled

Use one permanent Cloudflare API token with the scopes the project needs, stored once in the secure credential store. Never roll it, never delete it, never ask for a pasted or transient token. If a future project needs a scope the token lacks, raise it before starting that project, never in the middle of the build. A token that is rolled mid-project breaks every deploy that depends on it.

### Rule 6: Bump the app version query string on every app.js change, and verify live after every deploy

Browsers cache JavaScript by URL. If you change `app.js` without changing its version query string, browsers keep serving the old copy and your fix looks broken when it is not. Bump the version on every change, then verify live: the homepage returns 200, a tool page returns 200, and the new code is in the served file. A deploy is not done until the live check passes.

The premium deploy-check script catches this before you deploy: `python3 deploy-check.py wrangler.toml --dist build` scans the built HTML for script and stylesheet references and warns on any local `.js` or `.css` reference without a version query string. A reference counts as versioned when it has a query string, or when the filename carries a content hash. The check is a warning, never an error, because a hashed filename is a valid way to bust the cache too. Run it as part of the pre-deploy check, not after the deploy.

## Core Workflow

### 1. First setup (new project)

Follow the order in `references/setup-checklists.md`. The order is: domain and DNS, then the Pages project, then the Worker, then routes, then secrets, then Email Routing, then D1, then the live verification. Do not skip ahead. A Worker routed before its DNS exists, or a secret set before the Worker exists, wastes a round trip each time.

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
5. If a binding is missing after the deploy, redeploy with the full list. Do not patch bindings by hand in the dashboard and leave the deploy config wrong.

### 4. Adding a binding, secret, or cron trigger

1. Add it to the config file (wrangler.toml) first, so the next deploy carries it.
2. Set secrets with `wrangler secret put` or in the dashboard. Secrets are never written in a config file and never pasted into chat.
3. Redeploy with the full bindings list (Rule 2). Adding one binding is not a reason to drop the others.
4. Verify the new binding or trigger works live, and that the old ones still do.

### 5. Diagnosing a broken deploy

Do not guess. Open `references/error-diagnosis.md` and work the table in order: symptom, likely cause, check, fix. The most common causes, in order, are: the direct-upload route (Rule 1), a wiped binding (Rule 2), a stale cached script (Rule 6), a missing or wrong route, and a secret that was never set.

## Plan limits worth knowing

These are the Cloudflare plan limits as documented in October 2026. Confirm them on Cloudflare's pricing page if a deploy behaves differently, because limits change.

- Workers Free: 100,000 requests per day, 5 cron triggers, 50 subrequests per invocation, D1 100,000 row writes per day.
- Workers Paid ($5 per month): 10 million requests per month included, Browser Rendering. The Paid plan is only needed when the project needs Browser Rendering or the headroom.
- Cloudflare blocks outbound port 25 from Workers. A port checker will report SMTP port 25 as closed or filtered. That is the platform, not the target.
- Bot Fight Mode blocks automated capture services on the main domain. If an external screenshot or check service fails on the main domain but works on the Pages deployment domain, that is the likely cause.

## Gotchas

- Never write a secret, token, account ID, zone ID, or database ID into a config file, a skill file, or chat. Secrets are set with `wrangler secret put` or in the dashboard. IDs are placeholders in examples until the real values are put in the real config.
- Never deploy a Worker without a route and then wonder why the domain returns nothing. Without a route, the Worker only publishes to its workers.dev subdomain.
- Never trust a deploy you have not checked live. A successful command exit is not a successful deploy. Homepage 200, tool page 200, new code in the served file.
- Never run a build and a deploy at the same time (Rule 4). The failure looks like a deploy bug and wastes an hour.
- Never roll the Cloudflare token mid-project (Rule 5). If a token stops working, check the scopes first, then the credential store, then ask before creating a new one.
- A compatibility date in the future is invalid. Cloudflare rejects it. A date older than 2021 is stale enough to warn about.
- Cron triggers are limited to 5 on the Free plan. A sixth trigger silently does not run.
