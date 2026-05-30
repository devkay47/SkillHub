---
name: changelog-generator
description: Generates a structured CHANGELOG.md from Git commit history following the Keep a Changelog format. Use when the user asks to generate a changelog, update release notes, document what changed in a version, or prepare release documentation.
---

## Overview

This skill reads the Git commit history and produces a CHANGELOG.md following the Keep a Changelog format (keepachangelog.com), which is the most widely adopted changelog standard in open source. It uses Conventional Commits types to categorise changes automatically and rewrites terse developer shorthand into plain English that non-developers can read.

## Instructions

When this skill activates, first determine the scope. If the user specifies a version range such as "since v1.2.0" or "between v1.0 and v2.0", use `git log v1.0..v2.0 --oneline` for that range. If no range is specified, generate the full changelog from all tags, grouping commits by the version tag nearest to them.

Run `git log --pretty=format:"%H|%s|%an|%ad" --date=short` to get structured commit data. Parse each commit message to determine its Conventional Commits type and map it to a Keep a Changelog section as follows. The `feat` type maps to Added. The `fix` type maps to Fixed. The `refactor`, `perf`, `docs`, and `style` types map to Changed. Security-related fixes map to Security. Removal of features maps to Removed. Deprecation notices map to Deprecated. Ignore `test`, `ci`, and non-user-visible `chore` entries entirely — these are internal changes that do not belong in a user-facing changelog.

Format the CHANGELOG.md with the unreleased section at the top and oldest versions at the bottom. Each version section header uses the format `## [version] - YYYY-MM-DD`. The `[Unreleased]` section collects all commits since the most recent tag.

For each change entry, rewrite the commit subject line into plain English. "fix(auth): handle null session token" becomes "Fixed a crash that occurred when the session token was null on login." The changelog is read by users, not just developers — write for them.

After generating, present the changelog for review before writing it to disk. Always ask the user to confirm before saving.

## Keep a Changelog Format
