# Contributing to SkillHub

Thank you for considering a contribution to SkillHub. This document explains everything you need to know to submit a skill successfully — from understanding what makes a good skill, to creating the required files, to opening a Pull Request that passes review on the first attempt.

Read this document fully before opening a PR. Most submissions that are rejected on the first pass are rejected because of issues that are explicitly covered here.

---

## What SkillHub Is and Is Not

SkillHub is a **curated** registry. That word is doing real work. It means every skill listed here has been personally reviewed by a maintainer, tested to verify it works, and confirmed to meet the quality bar described in this document. There is no automated approval. There is no bulk import. Every single skill earns its place individually.

This is intentional. The existing marketplaces — SkillsMP, LobeHub, ClaudeSkills.info — have hundreds of thousands of skills between them, most of which are untested, undocumented, or simply do not work reliably. SkillHub exists precisely because there is no curated alternative. Maintaining that curation is the project's entire value proposition, and every contributor who submits a high-quality skill is directly protecting that value.

If you want to contribute, what you are really doing is raising the quality bar for the entire ecosystem. That is worth doing carefully.

---

## The Three Files Every Skill Must Have

Every skill submission consists of exactly one folder containing exactly three files. The folder name is the skill's unique identifier and must be lowercase letters and hyphens only — for example `git-commit-formatter` or `pdf-table-extractor`. No uppercase letters, no underscores, no spaces.

**File 1: SKILL.md**

This is the file that the AI agent actually reads and follows. It is the most important file in your submission. It must begin with YAML frontmatter — a block of structured metadata enclosed in triple dashes — followed by the skill's instructions written in plain markdown.

The YAML frontmatter has two required fields. The `name` field must be identical to your folder name. The `description` field is the routing mechanism — it is what the agent reads to decide whether to activate your skill for a given task. Writing a good description is the single most important thing you can do to make your skill work reliably.

A good description does three things simultaneously: it states what the skill does, it specifies the conditions under which the skill should activate, and it is written in third person so it integrates cleanly into the agent's system prompt. Here is an example of a weak description and a strong one:

Weak: `Helps with Git commits.`

Strong: `Generates structured conventional commit messages by analysing staged diffs. Use when the user asks for help writing a commit message, asks to commit changes, or has staged files ready to commit.`

The difference is specificity. The weak description gives the agent no information about when to activate the skill. The strong description tells the agent exactly what inputs trigger it and what it produces.

After the frontmatter, write your skill instructions in plain markdown. Be direct and imperative — tell the agent what to do, not what to consider doing. Use numbered steps for sequential processes. Include at least one concrete example showing the expected input and output. Keep the instructions under 5,000 tokens in total, as this is the recommended maximum for skill bodies to avoid context window pressure.

**File 2: README.md**

This is the human-readable documentation for your skill. It is what contributors, reviewers, and users read to understand what the skill does before installing it. It must cover four things: what the skill does in plain language, which agents it is compatible with and how to install it, any dependencies required (libraries, tools, environment variables), and at least one real example of the skill working correctly.

The README is distinct from the SKILL.md. The SKILL.md is written for the agent. The README is written for the developer. Do not mix them up.

**File 3: metadata.json**

This is SkillHub's registry metadata. It is not read by agents — it is read by the SkillHub website to enable filtering, searching, and compatibility matching. Every field must be filled in accurately.

```json
{
  "name": "your-skill-name",
  "description": "One sentence description of what this skill does.",
  "category": "coding-and-development",
  "compatible_with": ["claude-code", "cursor", "codex", "openclaw", "gemini-cli"],
  "author": "your-github-username",
  "version": "1.0.0",
  "verified": false,
  "tags": ["relevant", "specific", "tags"],
  "dependencies": [],
  "tested_on": ["claude-code", "openclaw"]
}
```
