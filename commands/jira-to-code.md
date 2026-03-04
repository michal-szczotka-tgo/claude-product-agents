---
name: jira-to-code
description: Full workflow from a Jira ticket to a merged pull request -- read the ticket, explore code, plan changes, implement, review, and create PR. Use when the user says "implement this ticket", "work on JIRA issue", "pick up this task", "jira to code", or provides a Jira ticket ID to implement.
---

# Jira to Code

Complete workflow that turns a Jira ticket into a shipped pull request.

## Workflow

### 1. Read the Ticket

Use the Jira MCP to fetch the full ticket:
- Summary, description, acceptance criteria
- Priority, story points, linked issues
- Comments for additional context

If anything is ambiguous, ask the user before proceeding.

### 2. Explore the Codebase

Before writing any code:
- Search for related files, modules, and patterns
- Query the database schema if the change touches data
- Identify dependencies and potential side effects
- Map out which files need changes

Summarize your understanding back to the user for confirmation.

### 3. Plan the Changes

Create a concise implementation plan:
- List each file to modify and what changes are needed
- Note any risks or edge cases
- Estimate complexity (small / medium / large)

Get user approval before proceeding.

### 4. Implement

- Follow existing code patterns and conventions
- Write minimal, clean code
- Handle errors and edge cases
- Add tests if the project has a test suite

### 5. Self-Review

Before creating the PR:
- Run the review checklist (logging, error handling, types, security)
- Check for debug statements and hardcoded values
- Ensure changes match the ticket requirements

### 6. Create PR

Use the `create-pr` skill to:
- Create branch, commit, push
- Write PR description referencing the Jira ticket
- Link the PR to the Jira issue

### 7. Update Jira

Transition the ticket status (e.g., In Progress -> In Review) and add a comment with the PR link.
