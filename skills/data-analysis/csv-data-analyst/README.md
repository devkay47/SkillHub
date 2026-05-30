# csv-data-analyst

Analyses CSV files methodically — starting with a data quality audit before producing any statistics, so you never get confident numbers from dirty data.

## What It Does

When you ask your agent to analyse or explore a CSV file, this skill activates. It runs a six-phase pipeline: load with defensive type detection, data quality assessment (missing values, duplicates, type inconsistencies), user-approved type conversion, statistical summary, outlier detection using the IQR method, and finally a plain-language summary of the three to five most interesting findings in the actual data.

The defensive approach is deliberate. The skill surfaces data quality problems before aggregating, rather than producing statistics that silently incorporate bad data. It pauses after the quality report and asks the user whether to proceed — this checkpoint prevents the common failure where analysis runs on a dataset with 30% missing values in a key column and produces averages that are technically computed but meaningfully wrong.

## Compatible Agents

Verified working on Claude Code, OpenClaw, Codex CLI, Cursor, and Gemini CLI.

## Installation

Copy the `csv-data-analyst` folder into your agent's skills directory.

For Claude Code (project scope): `.claude/skills/csv-data-analyst/`
For Claude Code (global): `~/.claude/skills/csv-data-analyst/`
For OpenClaw: `.openclaw/skills/csv-data-analyst/`
For Cursor: `.cursor/skills/csv-data-analyst/`

## Dependencies

Install with `pip install pandas numpy`.

## Example

**You say:** "Analyse this sales data CSV and tell me what it shows"

**Agent produces:**

A data quality report first — noting that the `revenue` column has 12% missing values concentrated in March and April, and that the `order_date` column has two inconsistent date formats across different rows. It asks whether to proceed. After confirmation, it produces a full statistical summary and closes with specific findings — for example that the top three regions account for 71% of total volume, and that 23 order values are statistical outliers likely representing enterprise contracts that should be segmented separately.

## Notes

The key findings section always produces specific observations about the actual values in your data — not generic statements. If the skill cannot identify anything genuinely interesting, it says so rather than padding the output with filler.
