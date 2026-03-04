---
name: post-refinement
description: Condense a verbose Jira ticket into a concise engineer-ready format after refinement. Archives the original description as a comment and replaces it with a tight 4-section summary. Use when the user says "clean up ticket", "post-refinement", "finalize ticket", "condense ticket", "make ticket concise", or after a refinement session is complete.
---

# Post-Refinement Ticket Cleanup

The ticket has been through creation (`create-jira-issue`), peer review, and refinement prep. It's bloated with context that was useful for discussion but slows down the implementing engineer. This skill distills it to what matters.

## Final Description Format

```
## TL;DR
[1-2 sentences: what changes and why]

## Acceptance Criteria
- [ ] AC1
- [ ] AC2
- [ ] AC3
(3-5 items. Testable. Unambiguous. No filler.)

## Technical Approach
- Change X in [file.php](https://github.com/TransferGo/repo/blob/master/path)
- Add Y using pattern Z
(3-5 bullets. Only verified files -- never imagined paths.)

## Risks
- Risk description
(0-2 bullets. Omit section entirely if none.)
```

That's it. Four sections. Every line earns its place.

## Workflow

### Step 1: Read the ticket

Fetch via `jira_get_issue` with `fields: "*all"`. Read the description AND all comments (refinement prep notes, peer review findings, codebase scan results live there).

### Step 2: Archive the original description

Post a comment via `jira_add_comment`:

```
**Original description (pre-cleanup):**

[full current description here]
```

This preserves all context -- user stories, current vs desired state, detailed rationale -- for anyone who needs the backstory.

### Step 3: Verify files

For every file path that will appear in Technical Approach:
- Confirm it exists in the repo via GitHub MCP (`get_file_contents` or `search_code`)
- If a file doesn't exist, it's a proposed new file -- mark it clearly: "*(new file)*"
- Remove any file references that were speculative or incorrect

### Step 4: Distill

Pull from the full ticket history (description + comments) to build the 4-section format:

**TL;DR:** Collapse the User Story + Current/Desired State into 1-2 sentences. Lead with the change, follow with the why.

**Acceptance Criteria:** Take the existing ACs. If more than 5, consolidate related ones. If any are vague ("improve error handling"), rewrite them to be testable ("return JSON with `error_code`, `error_message`, `is_retriable` fields"). Incorporate any AC refinements from comments.

**Technical Approach:** Merge the Affected Files section with any architectural decisions captured in refinement comments. Each bullet = one concrete change with a verified file link. Do not list files that aren't changing.

**Risks:** Only include risks that would change how an engineer implements the ticket. "This is complex" is not a risk. "Requires Redis which isn't in the charge domain yet -- needs infra setup first" is.

### Step 5: Update the ticket

Replace the description via `jira_update_issue`.

## Rules

- **No new information.** You are a distiller, not a researcher. Everything in the final description must trace back to the existing description or comments.
- **3-5 ACs, hard limit.** If you can't fit it in 5, the ticket should be split.
- **Only real files.** Every GitHub link must be verified. One broken link destroys trust.
- **Omit sections with nothing to say.** If there are no risks, don't include a Risks section with "None."
- **Keep the tone neutral and direct.** No hedging ("we might want to consider..."). State what needs to happen.

## TransferGo Repo Links

Use the correct base URL when linking files:
- Payments gateway: `https://github.com/TransferGo/payments-gateway-service/blob/master/`
- Main backend: `https://github.com/TransferGo/transfergo/blob/master/`
- Payments settings: `https://github.com/TransferGo/payments-settings-service/blob/master/`
- Payments SDK: `https://github.com/TransferGo/payments-settings-sdk/blob/master/`
- Admin frontend: `https://github.com/TransferGo/admin-frontend-old/blob/master/`
