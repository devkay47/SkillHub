# wordpress-rescue

Diagnose and fix WordPress errors from the actual debug.log, with a safe rollback procedure.

## The problem

A WordPress site breaks and the owner guesses. A white screen could be a fatal error in a plugin, a memory limit, a broken theme, or a dead database, and each one has a different fix. Guessing from the symptom alone is how a small error becomes a broken site, because the wrong fix gets applied to the wrong cause, and there is no backup to undo it.

This skill teaches the agent to read the log first. The log names the file, the file names the plugin or theme, and the error class names the fix. Then it follows a safe rollback procedure: back up before any change, make one change at a time, verify after each, and roll back at once if the fix makes it worse.

## What is in the free skill

- `wordpress-rescue/SKILL.md`: the diagnosis-first workflow, the safe rollback procedure, the plugin and theme isolation sequences, a worked example, gotchas, and a troubleshooting table.
- `wordpress-rescue/references/error-decision-tree.md`: the full decision tree for ten errors (white screen, database connection, HTTP 500, redirect loops, plugin conflicts, theme conflicts, .htaccess corruption, PHP version and memory limits, file permissions, no log at all). Each entry has the log line, the most likely cause, the fix, and the rollback.
- `wordpress-rescue/INSTALL.md`: install steps for Claude Code, Codex CLI, Cursor, and Claude.ai.
- `wordpress-rescue/examples/`: four hand-written sample debug.log files (a fatal error in a plugin, a database connection error, a memory exhaustion, and a clean log with warnings only) plus a worked diagnosis of the fatal plugin sample.

## Install

```bash
mkdir -p ~/.claude/skills
cp -r wordpress-rescue ~/.claude/skills/
```

See `wordpress-rescue/INSTALL.md` for Codex, Cursor, and Claude.ai.

## Premium pack

The premium pack adds the tools that do the diagnosis for you:

- `diagnose.py`: a script that parses a WordPress debug.log, identifies the failing plugin or theme, the file and line, and the error class, and suggests the fix and the rollback step. Python 3, no dependencies. It also has a `--config` mode that scans `wp-config.php` (and a theme `functions.php`) for leading or trailing whitespace before `<?php` or after `?>`.
- `rescue.php`: a standalone run-once emergency file that disables all plugins and switches to a default theme. Upload it to the site root, visit it in a browser, confirm, then delete it immediately.
- `force-logging-snippet.php`: a drop-in snippet for `wp-config.php` that forces PHP errors into a custom log file regardless of server defaults.
- `rollback-checklist.md`: the checkbox version of the safe rollback procedure, to run before any fix.
- The error decision tree, included in the skill copy in this pack.

Price: $15, one-time payment, single-user licence. The premium pack is sold on websmartools.com. The free skill is MIT licensed.
