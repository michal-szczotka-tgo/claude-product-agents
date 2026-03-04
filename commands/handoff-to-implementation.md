---
name: handoff-to-implementation
description: Validate a PRD's completeness and generate a structured handoff summary for the payments-agents planner. Use when the user says "hand off", "send to implementation", "ready for engineering", "handoff", "ready for planner", or when Phase 5 of the CTO discovery pipeline is reached.
---

# Handoff to Implementation

Bridge between the discovery workspace and the payments-agents planner. Validates the PRD is complete, then generates a structured handoff that the planner can consume directly.

## When to Use

This skill runs at Phase 5 of the CTO discovery pipeline, after the PRD (Phase 3) and UX Review (Phase 4) are complete. It can also be triggered standalone when the PM has a finished PRD ready for engineering.

## Step 1: Locate the PRD

1. Ask the user for the Notion PRD URL (or use the one from the current conversation)
2. Fetch the PRD content via Notion MCP
3. If Notion MCP is unavailable, ask the user to paste the PRD content

## Step 2: Validate Completeness

Check that every required section exists and is non-empty:

```
PRD COMPLETENESS CHECK:
- [ ] 1. Problem Statement -- data-backed, not vague
- [ ] 2. Proposed Solution -- conceptual, not implementation-level
- [ ] 3. Success Metrics -- measurable, with current/target values
- [ ] 4. UX Impact Summary -- screens, states, gaps documented
- [ ] 5. Scope Boundaries -- in-scope and out-of-scope explicit
- [ ] 6. Business Flows & Scenarios -- at least one happy path fully mapped with transaction states
- [ ] 7. Edge Cases & Business Rules -- at least one edge case with all fields
- [ ] 8. Business Acceptance Criteria -- Given/When/Then for every flow in Section 6
- [ ] 9. Milestones / Phasing -- phases with estimates and dependencies
- [ ] 10. Open Questions
```

**If any section is missing or incomplete:**
- List the gaps
- Ask the PM: "Should I fill these in from our conversation, or do you want to complete them first?"
- Do NOT proceed to Step 3 until all sections pass

## Step 3: Generate Handoff Summary

Produce a structured document optimized for the payments-agents planner. The planner has these subagents: `@pgw` (payments-gateway-service), `@ps` (payments-settings-service), `@ps-sdk` (payments-settings-sdk), `@account-lib` (php-account-serialisation-lib), `@remittance` (transfergo monolith).

**Handoff format:**

```
# Implementation Handoff: [Feature Name]

## Problem
[1 paragraph from PRD Section 1]

## Service Ownership

| Feature Area | Service | Planner Subagent |
|---|---|---|
| [area] | [service-name] | @pgw / @ps / @remittance / etc. |

## What Each Service Needs to Do

### @pgw (payments-gateway-service)
[High-level: what PGW needs to handle, constraints]

### @ps (payments-settings-service)
[If applicable]

### @remittance (transfergo monolith)
[If applicable]

[Include only services that are in scope]

## Business Flows to Implement
[Reference the flows from PRD Section 6 -- the planner maps these to service-level work]

## UX Requirements
[From PRD Section 4 -- new states, screen descriptions, user flow]

## Phasing
[From PRD Section 9 -- phases with dependencies]

## Open Questions for Engineers
[From PRD Section 10]

## Risks
[Consolidated from the PRD]

## Source
- PRD: [Notion URL]
- Jira Story: [link if created]
- Discovery conversation: [date and brief description]
```

## Step 4: Deliver the Handoff

1. **If Notion MCP is available:** Add the handoff summary as a new section at the bottom of the PRD page, or create a linked child page titled "[Feature Name] -- Implementation Handoff"
2. **If Notion MCP is unavailable:** Output as structured markdown

Then tell the PM:
- "The handoff is ready. Open the payments-agents workspace and share this with the planner."
- "The planner will route work to the correct service subagents based on the Service Ownership table."
- Link to the Notion page

## Rules

- This skill is a **validator and formatter**, not a researcher. Do not explore code or generate new findings.
- Everything in the handoff must trace back to the PRD. If information is missing, flag it -- don't invent it.
- The handoff must map every feature area to a planner subagent (`@pgw`, `@ps`, `@ps-sdk`, `@account-lib`, `@remittance`).
- Keep the handoff concise. The planner has its own exploration capabilities -- it doesn't need exhaustive context, just clear direction.
