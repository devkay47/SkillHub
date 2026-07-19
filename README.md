# SkillHub 🧠

### The curated, model-agnostic marketplace for AI agent skills.
### Think npm — but for AI agents.

![Skills](https://img.shields.io/badge/skills-growing-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen)
![Model Agnostic](https://img.shields.io/badge/model-agnostic-orange)

---

## What Is SkillHub?

An AI agent "skill" is a structured instruction file — a `SKILL.md` — that teaches an AI coding agent how to perform a specific task properly. Instead of every developer spending hours prompting their agent from scratch to figure out how to create a Word document, parse a PDF, or integrate an API, they install a skill and the agent immediately knows exactly what to do.

SkillHub is the place where those skills live.

It is a **curated, community-driven registry** of high-quality, verified AI agent skills that work across all major agents — Claude Code, Cursor, Codex, OpenClaw, and any agent that reads instruction files. Every skill listed here has been reviewed, tested, and confirmed to work. No scraped junk. No untested experiments. Just skills that actually deliver results.

---

## Why SkillHub Exists

The AI agent skills ecosystem already has marketplaces. But every single one has a critical problem.

**Skills.sh** is community-driven but unreviewed — quality varies wildly and you have no way to know if a skill works before you install it. **SkillsMP** has indexed over 800,000 skills scraped from GitHub — a massive catalog with zero curation, where every skill requires your own audit before trusting it. **LobeHub** has 169,000+ skills but is deeply tied to its own ecosystem. **ClaudeSkills.info** has 658 community skills but limited discoverability and no structured submission process.

The pattern is clear: the existing marketplaces are Play Stores — open, unfiltered, and overwhelming. **SkillHub is the App Store.** Every skill is reviewed before it is listed. Every skill has documented compatibility. Every skill has a clear description of exactly what it does and how to use it.

Curation is the moat. It is the only thing the existing marketplaces cannot copy without rebuilding from scratch.

---

## How To Install a Skill

Installing a skill is a two-step process that takes about sixty seconds.

**Step 1** — Find the skill you want by browsing the categories below or searching the repo. Click into the skill folder and open the `SKILL.md` file.

**Step 2** - Copy the complete skill folder, not only `SKILL.md`, into your
agent's skill root. Keeping the folder intact preserves its README, metadata,
references, scripts, and assets.

```bash
# Claude Code project scope
cp -R skills/<category>/<skill-name> .claude/skills/<skill-name>

# Cursor project scope
cp -R skills/<category>/<skill-name> .cursor/skills/<skill-name>

# OpenClaw workspace scope
cp -R skills/<category>/<skill-name> skills/<skill-name>
```

That is it. Restart or reload the agent after installing the folder. The agent
will then use the skill when its activation description matches the task.

---

## The Six Categories

SkillHub launches with six carefully chosen categories. Each one represents a major area where developers consistently need their agents to perform better. The categories are broad enough to feel comprehensive from day one, but focused enough that every skill listed clearly belongs.

**Coding and Development** contains skills that improve how agents write, review, refactor, and test code. These are the highest-demand skills in the entire registry and the category most likely to produce viral single skills.

**Document Processing** contains skills for working with PDFs, Word documents, spreadsheets, and presentations. Agents notoriously struggle with file format nuances — these skills encode the hard-won knowledge of exactly how to produce professional output in each format.

**Data Analysis** contains skills for working with CSV files, databases, and data visualisation. These skills teach agents how to approach analytical tasks methodically rather than generating plausible-looking but incorrect results.

**API Integration** contains skills for working with external APIs and web services. These skills encode authentication patterns, error handling best practices, and format-specific quirks so agents can integrate APIs reliably.

**Agent Memory and Context** contains skills that help agents maintain context and remember information across sessions. This is one of the most underserved categories in existing marketplaces and one of the highest-value areas for serious agent workflows.

**Productivity Automation** contains skills for automating repetitive developer workflows. From CI/CD pipeline management to commit message generation to automated documentation, these skills handle the tasks developers do every day but should not have to think about.

---

## Repository Structure

Every skill in SkillHub follows an identical folder structure. This is not a suggestion — it is a requirement. The consistent structure is what allows the SkillHub website to read, display, and filter skills programmatically without any manual work.

```
skills/
├── coding-and-development/
│   ├── skill-name/
│   │   ├── SKILL.md          ← the actual skill instructions for the agent
│   │   ├── README.md         ← human-readable explanation and install guide
│   │   └── metadata.json     ← category, compatibility, author, version
├── document-processing/
│   └── ...
├── data-analysis/
│   └── ...
├── api-integration/
│   └── ...
├── agent-memory-and-context/
│   └── ...
└── productivity-automation/
    └── ...
```

---

## The Metadata Standard

Every skill must include a `metadata.json` file with the following fields. This is what powers filtering, search, and compatibility matching on the SkillHub website.

```json
{
  "name": "Skill Name",
  "description": "One sentence description of what this skill does.",
  "category": "coding-and-development",
  "compatible_with": ["claude-code", "cursor", "codex", "openclaw"],
  "author": "your-github-username",
  "version": "1.0.0",
  "verified": true,
  "tags": ["relevant", "tags", "here"]
}
```

The `verified` field is set to `true` only by SkillHub maintainers after the skill has been reviewed and tested. When you submit a skill via Pull Request, set this to `false` — a maintainer will update it after review.

---

## How To Submit a Skill

SkillHub grows because developers like you contribute skills. The entire submission process happens through GitHub Pull Requests, which means you get full credit for your contribution, your GitHub profile is linked to your skills forever, and the community can see your work.

Here is the step-by-step process.

First, fork this repository to your own GitHub account by clicking the Fork button at the top right of this page. This creates your own copy of the repo that you can edit freely.

Second, inside your fork, create a new folder inside the appropriate category directory. Name the folder after your skill using lowercase letters and hyphens — for example, `pdf-table-extractor` or `git-commit-formatter`.

Third, create the three required files inside your skill folder: the `SKILL.md` with your actual skill instructions, the `README.md` explaining what the skill does and how to install it, and the `metadata.json` with the standard fields filled in.

Fourth, open a Pull Request from your fork back to this repo. The PR template will guide you through a checklist that ensures your submission has everything it needs. A maintainer will review your skill within 48 hours, leave any feedback directly in the PR, and merge it once it meets the quality bar.

If your skill is accepted, it gets a **Verified** badge and is immediately live in the marketplace.

---

## The Verified Badge

The Verified badge is SkillHub's quality signal. It means a maintainer has personally tested the skill, confirmed it works as described, checked that the metadata is accurate, and determined that the instructions are clear enough for a developer to install and use without needing additional help.

Not every submitted skill will be verified immediately. Some skills will be approved with feedback and improvement suggestions. Some will be declined with a clear explanation of why. The review process is what makes SkillHub worth using — if every skill were automatically verified, the badge would mean nothing.

---

## What Makes a Great Skill

The best skills in SkillHub share a few characteristics that are worth understanding before you write your first one.

A great skill solves a **specific, named problem** that developers encounter regularly. "Write better code" is not a skill — "generate JSDoc comments for TypeScript functions" is a skill. The more specific the problem, the more useful the skill, because the agent can apply it precisely rather than broadly.

A great skill encodes **non-obvious knowledge**. If a developer could get the same result by typing a simple prompt themselves, the skill does not add much value. The best skills encode the hard-won knowledge that comes from hours of experimentation — the edge cases, the format quirks, the error handling patterns that are not obvious from first principles.

A great skill is **agent-agnostic in its instructions**. It does not reference any specific agent by name inside the `SKILL.md` instructions themselves. The metadata file handles compatibility — the skill instructions should work the same way regardless of which agent reads them.

---

## Comparison With Existing Marketplaces

| | SkillHub | Skills.sh | SkillsMP | LobeHub |
|---|---|---|---|---|
| Curated & Reviewed | ✅ Every skill | ❌ No review | ❌ No review | ❌ No review |
| Model Agnostic | ✅ All major agents | ⚠️ Partial | ⚠️ Partial | ❌ LobeHub only |
| Verified Badge | ✅ Yes | ❌ No | ❌ No | ❌ No |
| Community PR Submissions | ✅ Yes | ✅ Yes | ❌ Scraped | ✅ Yes |
| Open Source | ✅ MIT | ✅ Yes | ❌ No | ⚠️ Partial |

---

## Contributing

Read the [CONTRIBUTING.md](./CONTRIBUTING.md) for the full guide. The short version: fork, create your skill folder with three files, open a Pull Request, and a maintainer will review within 48 hours.

All contributions are welcome — new skills, improvements to existing skills, fixes to documentation, and suggestions for new categories.

---

## License

MIT License — see [LICENSE](./LICENSE) for details. Every skill in this registry is freely usable, modifiable, and redistributable. The only requirement is attribution when redistribution is involved.

---

## Star History

If SkillHub is useful to you, please give it a star ⭐ at the top of this page. Stars help other developers discover the project, which brings in more contributors, which means more high-quality skills for everyone. It is the simplest thing you can do to help the project grow.

---

*SkillHub — The only curated, model-agnostic, community-driven AI agent skills marketplace.*
