# Agent Memory and Context Skills

This category contains skills that help AI agents maintain useful context across sessions, manage what they remember and when, and avoid the common failure mode of starting every task as if it were the first time the agent has encountered the project.

## What Belongs Here

A skill belongs in this category if its primary value is improving how an agent handles persistent knowledge, session continuity, or contextual awareness. This includes skills for capturing and structuring learnings from a session so they survive into the next one, skills for building and maintaining a project knowledge base the agent can reference, skills for tracking decisions and their rationale over time, and skills for managing context window usage efficiently so the agent does not lose important information mid-task.

If the skill is primarily about automating a workflow, it belongs in Productivity Automation. If it is about writing better code in general, it belongs in Coding and Development. The question to ask: is the core value of this skill about what the agent remembers and how it uses that memory? If yes, it belongs here.

## Why This Category Matters

Memory and context management is the most underserved category in the entire AI agent skills ecosystem. Most developers experience this failure mode directly — they spend twenty minutes establishing context with an agent, the session ends, and the next session starts cold with no memory of what was discussed. The skills in this category exist to solve this problem in practical, implementable ways. They are not theoretical — they define specific file formats, update schedules, and retrieval patterns that agents can follow consistently.

## Quality Bar For This Category

Memory skills must be concrete about the format they use to store information. A skill that says "remember useful things" without specifying what file to write to, in what format, and when to update it cannot be verified. Reviewers will test whether the skill produces a consistent, readable output that persists across sessions and that a subsequent session can actually use.

## Skills In This Category

| Skill | Description | Compatible With |
|---|---|---|
| `session-learning-tracker` | Captures errors, corrections, and discoveries during a session and writes them to a structured LEARNINGS.md file for future sessions | Claude Code, Codex, OpenClaw, Cursor |

## Installing a Skill From This Category

Copy the skill folder into your agent's skills directory. For Claude Code and OpenClaw, use `.claude/skills/` or `.openclaw/skills/` inside your project, or `~/.claude/skills/` for global personal use. Memory skills are most effective when installed at the project scope rather than globally, since the context they build is project-specific.

## Submitting a Skill To This Category

Read the main [CONTRIBUTING.md](../../CONTRIBUTING.md) before submitting. Memory skills must clearly specify the output file format, when the agent should write to it, and how a subsequent session should load and use the stored information. All three of these must be present for a skill to pass review.
