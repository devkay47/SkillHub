# email-deliverability

A Claude Code skill that checks email deliverability with live DNS lookups for SPF, DKIM and DMARC.

## What it does

- Checks **SPF** records for a domain, validates syntax, and flags common issues like missing records, too many DNS lookups, and overly permissive policies.
- Checks **DKIM** by looking up a domain's DKIM selectors (common selectors like `default`, `google`, `mail`, `s1`, `s2`, and more) and validates the public key.
- Checks **DMARC** policies, including `p=none`, `p=quarantine`, and `p=reject`, and evaluates alignment with SPF and DKIM.
- Produces a clear **deliverability score** (0–100) with a plain-language explanation of what is broken and how to fix it.

## Prerequisites

- Python 3.10+
- `pip install dnspython`

## Usage

```bash
python scripts/check_deliverability.py example.com
```

Or, in Claude Code, ask:

> Check email deliverability for example.com

and the skill will run the DNS lookups, validate SPF/DKIM/DMARC, and report the findings.

## Installation

Copy the `email-deliverability` folder into your skills directory:

- Claude Code: `~/.claude/skills/email-deliverability`
- Cursor: `~/.cursor/skills/email-deliverability`
- Codex: `~/.codex/skills/email-deliverability`

Then restart your agent if needed.

## Files

- `SKILL.md` — The skill instructions.
- `scripts/check_deliverability.py` — The deliverability checker script.
- `examples/example_output.md` — Example output for a domain.

## License

This skill is free to use. See the [SkillHub repository](https://github.com/devkay47/SkillHub) for details. It is not open source.
