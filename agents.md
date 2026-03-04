# Agent Reference

Full reference for all 19 Claude Code commands in this toolkit. Organized by pipeline stage.

---

## Stage 1 — Discovery & Product

---

### `/cto-partner`

**What it does**: Full pipeline orchestrator. Acts as CTO, running discovery from raw idea to Jira-ready tickets. Chains through all downstream skills automatically.

**Trigger phrases**: "CTO", "brainstorm", "let's discuss", "I have an idea", "new feature", "I found a bug"

**Inputs**: A product idea, user problem, bug report, or data observation

**Pipeline**:
```
Step 1: Brainstorm / understand the problem
Step 2: Ask clarifying questions until scope is clear
Step 3: Invoke /product-doc → creates PRD on Notion
Step 3a: Invoke /ux-review → UX impact analysis
Step 4: Invoke /explore → codebase exploration across all repos
Step 4a–4g: Mandatory exploration checklist (ownership, patterns, risks, complexity)
Step 4h: Surface considerations & open questions table (blockers vs. considerations)
Step 5: Phased implementation plan
Step 6: Create Jira tickets via Atlassian MCP
Step 7 (optional): PR execution via Cursor prompts
```

**Output**: PRD on Notion + UX Impact Summary + phased plan + Jira tickets with technical context comments

**Calls**: `/product-doc`, `/ux-review`, `/explore`

**Called by**: User directly

**Notes**: Stack-specific. Edit the "Architecture Boundaries" section for your repos and domain ownership.

---

### `/product-doc`

**What it does**: Creates high-quality product documents at executive review standard — PRDs, RCA reports, product concepts, technical specs.

**Trigger phrases**: "write a PRD", "create product doc", "draft a spec", "write documentation", "product concept", "RCA report"

**Inputs**: Feature description, problem statement, or discovery notes from `/cto-partner`

**Output**: Structured PRD published to Notion, covering: problem statement, proposed solution, success metrics, UX impact, scope boundaries, technical constraints, phasing, open questions

**Called by**: `/cto-partner` Step 3

---

### `/slack-to-jira`

**What it does**: Converts a flagged Slack message or conversation into a structured Jira RL ticket, fast.

**Trigger phrases**: "slack to jira", "triage this", "ticket this message", "create ticket from slack", pastes a Slack message or URL

**Inputs**: Slack message content or URL

**Output**: Jira RL ticket with TL;DR, current state, desired state, acceptance criteria, affected areas

**Notes**: Designed for quick triage — no codebase exploration. Use when a message lands in Slack that needs to become a ticket without a full discovery cycle.

---

### `/create-jira-issue`

**What it does**: Captures bugs, features, and improvements as structured Jira issues mid-development. Faster than `/cto-partner` — no PRD or full pipeline, just the ticket.

**Trigger phrases**: "create issue", "log a bug", "new ticket", "track this", "file a feature request"

**Inputs**: Issue description (bug or feature), current vs desired behavior

**Output**: Jira RL ticket with: TL;DR, current/desired state, acceptance criteria (with edge cases), conceptual approach, affected areas (GitHub-linked), dependencies, risks

**Calls**: Atlassian MCP for ticket creation; Grep/search tools for codebase verification

**Notes**: Includes mandatory codebase search to verify claims before writing the ticket. Every factual claim gets a confidence marker: `[VERIFIED]`, `[UNVERIFIED]`, or `[FROM PARTNER DOCS]`.

---

## Stage 2 — UX & Design

---

### `/ux-review`

**What it does**: Reviews UX impact of a proposed feature. Checks Figma for existing screens, identifies new states needed (loading, error, empty, blocked, partial), flags gaps, and optionally generates Figma prototypes.

**Trigger phrases**: "UX review", "check the UX", "what does the user see", "UX impact"

**Inputs**: Feature/change description, PRD, or Jira ticket draft

**Pipeline**:
```
Step 1: Check existing Figma designs (via Figma MCP)
Step 2: Analyze UX impact — what the user sees today vs. what changes
Step 3: Flag gaps (GAP / OK / QUESTION format)
Step 4: Ask PM if Figma prototypes are needed → delegates to /figma-prototype
Step 5: Verify spec against prototype — resolve [CONFIRM] annotations
Step 6: Update PRD with UX Impact Summary
```

**Output**: UX Impact Summary section for PRD + Figma prototype (if approved) + resolved spec annotations

**Calls**: `/figma-prototype` (if PM approves prototype creation)

**Called by**: `/cto-partner` Step 3a

---

### `/figma-prototype`

**What it does**: Generates a ready-to-run Figma plugin (`code.js` + `manifest.json`) from a UX spec file. Reads brand tokens from a reference Figma file via REST API, then generates all screens with exact copy, layout, and wired prototype connections.

**Trigger phrases**: "figma prototype", "create Figma screens", "generate prototype", "build Figma frames", "prototype this"

**Inputs**:
- UX spec file path (markdown with screen list, copy, flow connections)
- Figma reference file key (for brand token extraction)
- Figma PAT (`X-Figma-Token`)
- Feature slug (for output directory naming)
- Output Figma file URL (for run instructions)

**Output**: `product-docs/{feature-slug}-figma-plugin/code.js` + `manifest.json` + run instructions

**Why a plugin**: Figma REST API and MCP are read-only for design nodes. The Plugin API is the only supported path to programmatically create frames. Requires one manual run in Figma desktop (~30 seconds).

**Called by**: `/ux-review` Step 4

**Template reference**: `product-docs/partial-auth-figma-plugin/` — structural reference for generated plugins

---

## Stage 3 — Engineering Exploration

---

### `/explore`

**What it does**: Deep codebase analysis before implementation. Identifies dependencies, edge cases, existing patterns, and ambiguities. Runs across all repos in the workspace.

**Trigger phrases**: "explore", "analyze the codebase", "understand this first", "initial exploration"

**Inputs**: Feature description or Jira ticket

**Output**: File-level dependency map, request flow traces, reusable infrastructure found, risks, edge cases, complexity estimate

**Called by**: `/cto-partner` Step 4

---

### `/db-explore`

**What it does**: Understands the Redshift data warehouse schema — tables, columns, relationships — using exported DDL files.

**Trigger phrases**: "show me the database", "what tables", "db schema", "explore database"

**Inputs**: None required — reads DDL export files from `database-context/` directory

**Output**: Schema summary, relevant tables for a feature area, relationship map

---

## Stage 4 — Planning & Tickets

---

### `/create-plan`

**What it does**: Generates a structured markdown implementation plan with progress tracking, task breakdown, and critical decisions.

**Trigger phrases**: "create plan", "write a plan", "make a plan", "implementation plan"

**Inputs**: Exploration output from `/explore`, feature requirements

**Output**: Markdown plan file with phases, tasks, critical files, and verification steps

**Called by**: Typically after `/explore` in the `/cto-partner` pipeline

---

### `/refinement-prep`

**What it does**: Prepares for engineering refinement sessions by analyzing Jira tickets, anticipating engineer questions, and generating data-backed talking points.

**Trigger phrases**: "refinement", "refine tickets", "prep for refinement", "sprint planning", "prepare for grooming"

**Inputs**: Jira ticket(s) to refine

**Output**: Talking points per ticket, anticipated engineer questions with pre-answers, data points to pull before the session

---

### `/handoff-to-implementation`

**What it does**: Validates PRD completeness and generates a structured handoff summary for an engineering agent or planner. Phase 5 of the CTO discovery pipeline.

**Trigger phrases**: "hand off", "send to implementation", "ready for engineering", "handoff", "ready for planner"

**Inputs**: PRD (Notion page or markdown)

**Output**: Validated handoff summary with service ownership map, open questions flagged, phasing confirmed

---

## Stage 5 — Execution & Review

---

### `/execute`

**What it does**: Implements a feature precisely as planned — writes clean, modular code and tracks progress against a plan file.

**Trigger phrases**: "execute", "implement the plan", "build it", "start building", "go ahead and implement"

**Inputs**: Implementation plan (from `/create-plan`), Jira ticket context

**Output**: Code changes with status report per phase

---

### `/jira-to-code`

**What it does**: Full workflow from a Jira ticket to a merged pull request — reads ticket, explores code, plans changes, implements, reviews, and creates PR.

**Trigger phrases**: "implement this ticket", "work on JIRA issue", "pick up this task", "jira to code", Jira ticket ID

**Inputs**: Jira ticket ID or URL

**Output**: Implemented changes + GitHub PR

---

### `/create-pr`

**What it does**: Creates a GitHub pull request end-to-end — branch creation, commits, push, and structured PR description.

**Trigger phrases**: "create PR", "open pull request", "submit changes", "push my changes"

**Inputs**: Current branch state with committed changes

**Output**: GitHub PR with structured description linking to Jira ticket

---

### `/review`

**What it does**: Comprehensive code review covering PHP (Laravel, Symfony) and TypeScript/React patterns. Checks correctness, security, test coverage, and architectural alignment.

**Trigger phrases**: "review this code", "code review", "check this PR", "review my changes"

**Inputs**: PR URL, diff, or specific files

**Output**: Structured review with severity-rated findings (critical / major / minor / suggestion)

---

### `/peer-review`

**What it does**: Critically evaluates peer review findings from another team lead or model by verifying each issue against actual code. Prevents acting on hallucinated or incorrect review feedback.

**Trigger phrases**: "peer review", "review these findings", "evaluate this feedback", "check these comments"

**Inputs**: External review findings (pasted text or PR comments)

**Output**: Verified assessment — confirmed, rejected, or needs-more-context per finding

---

### `/post-refinement`

**What it does**: Condenses a verbose Jira ticket into a concise engineer-ready format after a refinement session. Archives the original description as a comment.

**Trigger phrases**: "clean up ticket", "post-refinement", "finalize ticket", "condense ticket", "make ticket concise"

**Inputs**: Jira ticket URL or ID

**Output**: Ticket updated in place with a 4-section summary (TL;DR, AC, approach, affected areas)

---

## Utilities

---

### `/learning-opportunity`

**What it does**: Pauses development to teach a concept at three increasing complexity levels: introductory, intermediate, and deep-dive.

**Trigger phrases**: "explain this", "teach me", "how does this work", "learning opportunity", "help me understand"

**Inputs**: Concept, pattern, tool, or architecture decision to explain

**Output**: Three-level explanation with code examples

---

### `/document`

**What it does**: Updates project documentation after code changes — verifies the actual implementation and updates CHANGELOG and docs to match.

**Trigger phrases**: "update docs", "document this", "update documentation", "add changelog entry"

**Inputs**: Recently completed feature or bugfix

**Output**: Updated CHANGELOG + relevant documentation files

---

### `/keybindings-help`

**What it does**: Configures Claude Code keyboard shortcuts — rebinds keys, adds chord bindings, modifies `~/.claude/keybindings.json`.

**Trigger phrases**: "rebind ctrl+s", "add a chord shortcut", "change the submit key", "customize keybindings"

**Inputs**: Desired keybinding change

**Output**: Updated `~/.claude/keybindings.json`

---

## Pipeline Chaining Reference

```
/cto-partner
  └─► /product-doc          (Step 3 — creates PRD)
  └─► /ux-review            (Step 3a — UX analysis)
        └─► /figma-prototype (Step 4 — if prototypes approved)
  └─► /explore              (Step 4 — codebase exploration)
  └─► /create-jira-issue    (Step 6 — tickets created)

/jira-to-code
  └─► /explore              (internal)
  └─► /create-pr            (internal)

/ux-review
  └─► /figma-prototype      (Step 4 — delegates prototype creation)
```

Commands not in the chain above are invoked directly by the user.
