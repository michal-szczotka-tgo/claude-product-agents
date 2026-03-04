---
name: explore
description: Analyze and understand a codebase before implementation, identifying dependencies, edge cases, and ambiguities. Use when the user says "explore", "analyze the codebase", "understand this first", "initial exploration", or wants to investigate how a feature integrates before building it.
---

# Initial Exploration Stage

Your task is NOT to implement yet, but to fully understand and prepare.

## Where Your Findings Go

Your exploration output is for **context enrichment**, not implementation prescription. Findings feed two artifacts:

1. **PRD update on Notion** -- conceptual findings, architecture boundaries, risks, open questions. This is the high-level view for stakeholders and engineers.
2. **Jira comment per ticket** -- technical deep-dive with file paths, class names, existing patterns, reusable infrastructure. This is for engineers to reference during refinement.

**Your findings do NOT go into the Jira ticket body as implementation instructions.** The ticket body gets a conceptual approach (what needs to happen). Your file-level exploration goes into Jira comments.

## Step 0: Map the Domain Across Repos

Before diving into any single repo, determine which repos are involved. TransferGo is a multi-repo system -- features often span multiple services.

**Mandatory cross-repo check:**
1. List all repos in the workspace
2. For each, read `AGENTS.md` or `CLAUDE.md` to understand what it owns
3. Search for the feature keyword (e.g., "charge", "3ds", "webhook") across ALL repos to find where the logic lives
4. Build an **architecture boundary map** for this feature:

```
Who owns what:
- [repo-name]: owns [endpoint/workflow/entity] -- [file path]
- [repo-name]: owns [endpoint/workflow/entity] -- [file path]

Communication:
- [repo-A] calls [repo-B] via [HTTP/queue/Temporal] at [endpoint]
- [repo-B] sends webhooks to [repo-A] at [endpoint]
```

**This step prevents the #1 mistake: assigning work to the wrong repo or proposing something that already exists.**

## Step 1: Read the Repo's AI Config

For each relevant repo, check for these files in the repo root:
- `AGENTS.md` -- agent instructions, tech stack, conventions
- `CLAUDE.md` -- same purpose, different name
- `.cursor/rules/` -- project-specific Cursor rules

Read them. They contain stack details, architecture decisions, and coding conventions you must follow.

## Step 2: Understand the Structure

- Identify the main source directory (`source/src/`, `src/`, `apps/`)
- Map out the module/service structure relevant to the feature
- Check `composer.json` or `package.json` for dependencies
- Look at existing tests to understand testing patterns

## Step 3: Check for Existing Implementations

Before proposing any new code, search for what already exists:

- **Search ALL repos** for classes, endpoints, and services related to the feature
- **Check for similar patterns** -- if proposing rate limiting for charges, search for existing rate limiting (e.g., payout rate limiting) and note the pattern
- **Check for existing endpoints** -- grep for route definitions related to the feature
- **Document what you find**: "X already exists at [file path]" or "No existing implementation found for X after searching [repos searched]"

This step is mandatory. Skipping it leads to tickets that propose building things that already exist.

## Step 4: Find Related Code

- Search for files, classes, and functions related to the feature
- Trace the request flow: route -> controller -> service -> repository -> database
- For each hop, record the file path
- Check for existing similar implementations to follow as a pattern

## Step 5: Check Data Layer

- Identify relevant database tables (check migrations or Doctrine entities)
- Read `database-context/schema.sql` in the product workspace if available
- Note any foreign keys, indexes, or constraints

## Step 6: Identify Risks

- Dependencies on other services or repos
- Migration requirements
- Potential side effects on existing functionality
- Performance implications at TransferGo's scale
- Business risks (liability shifts, compliance, chargeback exposure)

## Step 7: Report Back

Structure your findings into two outputs:

### Output A: For the PRD (conceptual, no file paths in this section)

**Architecture boundary map:**
- Which repo owns what (by domain, not by file)
- How they communicate

**What needs to change and where (conceptual):**
- Which services/domains are affected
- What data flows change
- What new capabilities are needed

**Risks and open questions:**
- Technical risks
- Business risks
- Things you could NOT verify (mark as `[UNVERIFIED]`)

### Output B: For Jira Comments (technical, file paths welcome)

**Existing implementations:**
- What already exists that's relevant, with file paths
- What can be reused, with file paths and pattern descriptions

**Detailed request flow:**
- Route -> controller -> service -> repository -> partner API, with file paths for each hop

**Recommended patterns to follow:**
- Only when backed by code citations
- Reference existing patterns with file paths

List all questions or ambiguities you need clarified. Do NOT assume requirements or scope beyond what was explicitly described. Do NOT fabricate technical details -- if you haven't seen it in the code, say "I don't know" and flag it as needing engineer input. Go back and forth until you have no further questions.
