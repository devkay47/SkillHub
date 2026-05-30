# Data Analysis Skills

This category contains skills that teach AI agents how to approach data problems methodically — working with CSV files, querying databases, cleaning messy data, and producing visualisations that are accurate and readable.

## What Belongs Here

A skill belongs in this category if its primary value is helping an agent handle structured data correctly. This includes reading and cleaning CSV or JSON data files, writing SQL queries, generating charts or graphs, performing statistical summaries, identifying outliers or anomalies, and transforming data between formats for downstream use.

If the skill is primarily about producing a formatted Excel or spreadsheet file as the final output, it may belong in Document Processing. If it is about connecting to a live database or external data API, it may belong in API Integration. The question to ask: is the core challenge in this skill about understanding and transforming data? If yes, it belongs here.

## Why This Category Matters

Data analysis is one of the areas where agents produce the most confidently wrong output. Without clear instructions, an agent analysing a CSV file may silently drop rows with missing values, misinterpret column types, or produce a chart whose axes are technically correct but visually misleading. The skills in this category encode defensive, methodical approaches to data work — the kind of habits a careful data engineer would follow.

## Quality Bar For This Category

Data analysis skills must specify what happens when the data is dirty — missing values, unexpected types, duplicate rows. A skill that only handles clean data is incomplete. Reviewers will test skills against datasets with intentional problems and expect the skill to handle them gracefully rather than crashing or silently producing wrong output.

## Skills In This Category

| Skill | Description | Compatible With |
|---|---|---|
| `csv-data-analyst` | Analyses CSV files with automatic type detection, missing value handling, and statistical summaries | Claude Code, Codex, OpenClaw, Cursor |

## Installing a Skill From This Category

Copy the skill folder into your agent's skills directory. For Claude Code and OpenClaw, use `.claude/skills/` or `.openclaw/skills/` inside your project, or `~/.claude/skills/` for global personal use.

## Submitting a Skill To This Category

Read the main [CONTRIBUTING.md](../../CONTRIBUTING.md) before submitting. Data analysis skills must include a sample dataset in the `examples/` directory so that reviewers can run the skill against real data during verification.
