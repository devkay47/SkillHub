# email-deliverability

Build and check SPF, DKIM, and DMARC records for a domain, using live DNS lookups.

## The problem

An AI agent writing email authentication records from memory gets them wrong in the same ways, every time:

1. It trusts a pasted record that is old or from a different domain, and judges a setup that does not exist.
2. It adds one provider too many, and the SPF record goes over the 10-lookup limit and fails outright at receivers.
3. It writes a DMARC record with no `rua` address, so nobody ever sees what fails.
4. It jumps straight to `p=reject`, and legitimate mail starts bouncing on day one.
5. It uses `+all` or the retired tags `pct`, `rf`, and `ri`, and the record is broken or out of date.
6. It declares a record correct while SPF passes and DMARC still fails, because alignment was never checked.

This skill teaches the agent the hard rules that keep a record correct: live DNS only, the lookup budget, alignment, and the safe rollout from monitoring to enforcement. The provider lookup costs come from live measurement, not from memory.

## What is in the free skill

- `email-deliverability/SKILL.md`: the hard rules, how to build SPF and DMARC records, the alignment rules, the rollout stages, a worked example, and a troubleshooting table.
- `email-deliverability/references/provider-includes.md`: the 11 measured provider includes with their lookup costs, what each one covers, and the alignment note for each provider.
- `email-deliverability/INSTALL.md`: install steps for Claude Code, Codex CLI, Cursor, and Claude.ai.
- `email-deliverability/examples/example-com-records.txt`: the worked example, SPF and DMARC records for a domain on Google Workspace and SendGrid.
- `email-deliverability/examples/build-checklist.md`: the checklist to run every time a record is built or changed.

## Install

```bash
mkdir -p ~/.claude/skills
cp -r email-deliverability ~/.claude/skills/
```

See `email-deliverability/INSTALL.md` for Codex, Cursor, and Claude.ai.

## Premium pack

The premium pack adds three things the free skill does not have:

- `check.py`: a script that fetches the live SPF, DKIM, and DMARC records for a domain and reports what is wrong. It counts SPF lookups including nested includes, flags `+all`, over-limit records, and missing records, checks the DMARC policy and retired tags, and checks that a DKIM key record exists for the selector. Python 3, standard library only, no dependencies.
- `references/record-templates.md`: ready-to-publish record templates for all 11 providers, each with its SPF include, its DKIM setup, and its alignment note.
- `dmarc-rollout-checklist.md`: the full DMARC enforcement rollout checklist, with the monitoring steps and what to check at each stage from `none` to `quarantine` to `reject`.

Get the premium pack at https://websmartools.com/email-deliverability/ for $15, one-time payment, single-user licence.

## The free web tool

A free SPF and DMARC generator already exists at https://websmartools.com/spf-dmarc-generator/. It builds both records in the browser, with a live lookup counter. This skill complements it: the tool builds the records, and the skill teaches Claude Code to build, check, and roll them out.

## Licence

The free skill is MIT licensed. See LICENSE. The premium pack has its own single-user licence and is not open source.
