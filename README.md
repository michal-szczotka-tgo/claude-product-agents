# Claude Product Agents

> 19 Claude Code slash commands covering the full product-to-engineering pipeline — from idea discovery through PRD, UX spec, Figma prototype generation, codebase exploration, Jira tickets, and PR execution.

---

## Why This Exists

Product-to-engineering handoff is where velocity goes to die. Requirements are misunderstood, UX gaps are discovered late, engineers start without enough context, and tickets get refined three times before anyone writes a line of code.

This toolkit enforces a repeatable, opinionated process: every feature starts with the same discovery workflow, produces the same artifacts, and lands in engineering with enough context to ship without re-explaining.

Built on [Claude Code](https://claude.ai/code). Each slash command is a markdown file with a system prompt, trigger conditions, and step-by-step instructions. Drop them in `~/.claude/commands/` and they become available as `/command-name` in any Claude Code session.

---

## Install

```bash
# Clone
git clone https://github.com/michal-szczotka-tgo/claude-product-agents.git

# Install all commands
cp claude-product-agents/commands/*.md ~/.claude/commands/
```

Commands are immediately available in Claude Code as `/command-name`. No restart required.

---

## Pipeline Overview

See [WORKFLOW.md](WORKFLOW.md) for the full Mermaid diagram.

```
DISCOVERY  (/cto-partner orchestrates)
  ├── /product-doc      → PRD on Notion
  ├── /ux-review        → UX gap analysis
  │     └── /figma-prototype  → Figma plugin (if approved)
  └── /explore          → codebase analysis → open questions table

PLANNING
  /create-plan → /create-jira-issue → /refinement-prep → /handoff-to-implementation

EXECUTION
  /jira-to-code → /execute → /create-pr → /review → /post-refinement

QUICK PATHS  (bypass discovery)
  /slack-to-jira · /create-jira-issue

UTILITIES
  /learning-opportunity · /document · /keybindings-help · /db-explore · /peer-review
```

---

## Skills Quick Reference

### Discovery & Product

| Command | Trigger | Output |
|---|---|---|
| `/cto-partner` | "brainstorm", "new feature", "I have an idea", "let's discuss" | Full pipeline: PRD → UX → exploration → phased plan → Jira tickets |
| `/product-doc` | "write a PRD", "create product doc", "draft a spec" | Structured PRD on Notion at executive review standard |
| `/slack-to-jira` | "slack to jira", paste a Slack message, "triage this" | Structured Jira RL ticket from a Slack conversation |
| `/create-jira-issue` | "create issue", "log a bug", "new ticket", "track this" | Jira ticket with AC, conceptual approach, affected areas |

### UX & Design

| Command | Trigger | Output |
|---|---|---|
| `/ux-review` | "UX review", "check the UX", "what does the user see" | UX gap analysis + Figma prototype generation |
| `/figma-prototype` | "figma prototype", "generate prototype", "build Figma frames" | Ready-to-run Figma plugin (`code.js` + `manifest.json`) |

### Engineering Exploration

| Command | Trigger | Output |
|---|---|---|
| `/explore` | "explore", "analyze the codebase", "understand this first" | File-level dependency map, edge cases, reusable patterns |
| `/db-explore` | "show me the database", "db schema", "what tables" | Schema understanding from exported DDL files |

### Planning & Tickets

| Command | Trigger | Output |
|---|---|---|
| `/create-plan` | "create plan", "implementation plan", "write a plan" | Structured markdown plan with task breakdown |
| `/refinement-prep` | "refinement", "prep for refinement", "sprint planning" | Data-backed talking points for engineers |
| `/handoff-to-implementation` | "hand off", "ready for engineering", "ready for planner" | Structured handoff summary for engineering agents |

### Execution & Review

| Command | Trigger | Output |
|---|---|---|
| `/execute` | "execute", "implement the plan", "build it" | Clean implementation with progress tracking |
| `/jira-to-code` | "implement this ticket", "jira to code", Jira ticket ID | Full workflow: ticket → code → PR |
| `/create-pr` | "create PR", "open pull request", "push my changes" | Structured GitHub PR with description |
| `/review` | "review this code", "code review", "check this PR" | Comprehensive review: PHP, Symfony, Laravel, TS/React |
| `/peer-review` | "peer review", "evaluate this feedback", "check these comments" | Verified assessment of external review findings |
| `/post-refinement` | "clean up ticket", "post-refinement", "condense ticket" | Concise engineer-ready ticket replacing verbose original |

### Utilities

| Command | Trigger | Output |
|---|---|---|
| `/learning-opportunity` | "explain this", "teach me", "how does this work" | Concept explanation at 3 depth levels |
| `/document` | "update docs", "document this", "add changelog entry" | Updated CHANGELOG and project docs |
| `/keybindings-help` | "rebind ctrl+s", "add a chord shortcut", "customize keybindings" | Updated `~/.claude/keybindings.json` |

---

## Prerequisites

- [Claude Code](https://claude.ai/code) — required
- **Recommended MCPs** (several commands use these if available):
  - [Figma Dev Mode MCP](https://help.figma.com/hc/en-us/articles/35041888416791) — for `/ux-review` and `/figma-prototype`
  - Atlassian MCP — for `/create-jira-issue`, `/cto-partner`, `/jira-to-code`
  - Notion MCP — for `/product-doc`, `/cto-partner`
  - GitHub MCP — for `/create-pr`, `/review`

Commands degrade gracefully when MCPs are unavailable — they fall back to structured markdown output.

---

## Stack Context

Several commands reference a specific backend stack. Customise before use if your stack differs:

| Command | Stack-specific references | What to change |
|---|---|---|
| `/cto-partner` | Laravel monolith, Symfony microservices, Temporal.io, Checkout.com, repo names | Architecture Boundaries section |
| `/explore` | PHP, Symfony, Laravel patterns | Search heuristics and file path conventions |
| `/review` | PHP 8.4, Laravel 12, Symfony 6.4, TypeScript/React | Review checklist and language-specific rules |
| `/db-explore` | Amazon Redshift, DDL export files | Database type and schema location |

All other commands (discovery, planning, UX, Jira) are stack-agnostic.

---

## How It Works

Each command is a markdown file with:

```yaml
---
name: command-name
description: When to invoke this command (used by Claude for auto-suggestion)
---

# Command Title

[System prompt: role, context, rules]

## Step 1: ...
## Step 2: ...
```

Claude Code reads these files on startup. When you type `/command-name`, the markdown becomes the active system prompt for that session, driving Claude's behavior through the defined steps.

Commands can call other commands explicitly (e.g., `/cto-partner` triggers `/product-doc` at Step 3, `/ux-review` at Step 3a, `/explore` at Step 4).

---

## Contributing

1. Fork the repo
2. Add or edit a command in `commands/`
3. Follow the format: YAML frontmatter + numbered steps + clear trigger conditions
4. PR with a description of what the command does and when it fires

---

## License

MIT
