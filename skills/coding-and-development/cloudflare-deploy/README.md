# cloudflare-deploy

Set up and deploy a site on Cloudflare Pages and Workers without the failures that cost a day each.

## The problem

An AI agent deploying to Cloudflare from memory gets the same things wrong, every time:

1. It deploys a Pages site through a direct-upload route, and the site serves HTTP 500 on every path.
2. It redeploys a Worker without the full bindings list, and the metadata replace wipes the D1 database and rate limit bindings.
3. It never checks the compatibility date, and the config drifts away from the deploy tool.
4. It runs a build and a deploy at the same time, and the failure looks like a deploy bug.
5. It changes `app.js` without bumping the version query string, and browsers keep serving the old copy.

This skill teaches the agent the six hard rules that stop those failures, plus the setup checklists and the error-diagnosis guide. They were learned from real deployments, where each rule is a failure that already happened once.

## What is in the free skill

- `cloudflare-deploy/SKILL.md`: the six hard rules, the first-setup order, the Pages deploy procedure, the Worker redeploy procedure, how to add a binding, secret, or cron trigger, and how to diagnose a broken deploy.
- `cloudflare-deploy/references/setup-checklists.md`: the setup checklists for DNS, Pages, Workers, Email Routing, and D1.
- `cloudflare-deploy/references/error-diagnosis.md`: the error-diagnosis guide. Symptom, likely cause, check, fix.
- `cloudflare-deploy/INSTALL.md`: install steps for Claude Code, Codex CLI, Cursor, and Claude.ai.
- `cloudflare-deploy/examples/wrangler-example.toml`: a complete, good example Worker config. Copy its shape, replace every example value.
- `cloudflare-deploy/examples/deploy-config-example.json`: an example deploy config for the premium deploy-check script.
- `cloudflare-deploy/examples/deploy-checklist.md`: the pre-deploy checklist.

## Install

```bash
mkdir -p ~/.claude/skills
cp -r cloudflare-deploy ~/.claude/skills/
```

See `cloudflare-deploy/INSTALL.md` for Codex, Cursor, and Claude.ai.

## Premium pack

The premium pack adds five things the free skill does not have:

- `deploy-check.py`: a script that checks a `wrangler.toml` plus a deploy config before you deploy. It catches a missing project name, a missing main file, a missing or malformed compatibility date, a bad route, a binding the deploy config expects but the config does not declare, and a secret that is listed but never set. With `--dist <dir>` it also warns on script and stylesheet references without a version query string, so browsers do not keep serving the old copy after a change. Python 3, no dependencies.
- `references/full-stack-wrangler.toml`: a complete reference Worker config (D1, two rate limit bindings, cron trigger, route) with example values to replace.
- `references/github-actions-deploy.yml`: a GitHub Actions workflow template that deploys a Pages site and a Worker from the main branch, running the checker first as a gate.
- `references/monorepo-wrangler.toml`: a monorepo blueprint for a project with a Pages frontend and two Workers.
- `deploy-verification-checklist.md`: the post-deploy live verification steps, the checks that prove a deploy actually worked.

Get the premium pack at https://websmartools.com/cloudflare-deploy/ for $15, one-time payment, single-user licence.

## Licence

The free skill is MIT licensed. See LICENSE. The premium pack has its own single-user licence and is not open source.
