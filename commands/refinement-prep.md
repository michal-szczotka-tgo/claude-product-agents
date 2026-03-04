---
name: refinement-prep
description: Prepare for technical refinement sessions by analyzing Jira tickets, anticipating engineer questions, and generating data-backed talking points. Use when the user says "refinement", "refine tickets", "prep for refinement", "sprint planning", "prepare for grooming", "technical review prep", or needs to present tickets to the engineering team.
---

# Refinement Prep

The user (PM) is about to present Jira tickets to the engineering team. Your job is to make them the most prepared person in the room. Think like a tech lead who will challenge every ticket -- then arm the PM with answers.

## Inputs

The user provides one of:
- Jira ticket keys (e.g., `RL-8966`, `RL-8967`)
- Task group IDs from the RCA report (e.g., "T1", "T2.1", "Phase 1 tickets")
- "All backend tickets" / "all RL tickets" -- run against the full set

## Step 1: Gather Data

For each ticket:
1. **Fetch from Jira** via Atlassian MCP (`jira_get_issue` with `fields: "*all"`)
2. **Read RCA context** from `RCA_Report.md`:
   - Section 1.4 (consolidated summary table) for phase, dependencies, impact
   - Section 2 (forensic deep dives) for root cause data and evidence
   - Section 3 (recommendations) for the proposed approach
   - Section 4 (engineering checklist) for implementation details
3. **Verify affected files** exist in the correct repos using GitHub MCP or grep. Flag any dead links.
4. **Check the ticket against the `create-jira-issue` skill structure** -- is it missing a User Story, Acceptance Criteria, Dependencies section, or Affected Files?

## Step 2: Generate Per-Ticket Refinement Brief

For each ticket, produce:

### 2.1 Elevator Pitch
2 sentences. What this ticket does and why it matters. Use plain language -- this is how the PM opens the discussion.

Example: "Right now 13,893 card transactions per month hit retry loops because we don't track per-card failures. This ticket adds a Redis-backed throttle that blocks retries after 2 consecutive failures on the same card within 30 minutes."

### 2.2 Technical Approach
Summarize in engineer terms:
- Which service(s) / repo(s) change
- What the code path looks like (e.g., "new middleware in payments-gateway-service that checks Redis before forwarding to CKO")
- What patterns are used (e.g., "Redis TTL keys", "Symfony event subscriber", "new Doctrine entity")
- What does NOT change (scope boundaries)

### 2.3 Anticipated Engineer Questions

Generate 3-6 realistic questions per ticket, categorized:

**Scope**
- "Why not also do X?" / "Can we cut this to just Y?"
- "Is this one sprint or should we split it?"
- "Does this need to handle [edge case]?"

**Dependencies**
- "Does this really block/depend on [other ticket]?"
- "What if the partner (CKO/Nuvei/etc.) doesn't deliver on time?"
- "Can we decouple this from [other ticket]?"

**Risk**
- "What's the rollback plan?"
- "What if this breaks existing payment flows?"
- "Backward compatibility -- do we need a feature flag?"
- "What about database migrations?"

**Data**
- "Where does the [number] come from?"
- "How fresh is this data?"
- "Is [N] transactions statistically significant?"

**Alternatives**
- "Why not [simpler approach]?"
- "Have we considered [library/service]?"
- "Could we solve this at the gateway level instead?"

**Testing**
- "How do we test this without hitting production?"
- "Do we need integration tests with the gateway sandbox?"
- "What about load testing for the Redis layer?"

### 2.4 Prepared Answers

For each question in 2.3, provide a 1-3 sentence answer that:
- Cites specific numbers from the RCA (e.g., "21,690 ACS timeouts in the analysis period")
- References specific code paths or files when relevant
- Acknowledges unknowns honestly (e.g., "Scope TBD pending CKO -- we'll refine the AC after their response")
- Points to the dependency map when dependencies are questioned

### 2.5 Red Flags to Proactively Address

Things the PM should raise FIRST, before engineers find them:
- Unknowns (e.g., partner-dependent scope)
- Low-confidence estimates
- Thin ticket descriptions that need team input
- Cross-repo coordination requirements
- Missing test infrastructure

Format: bullet list with a one-line explanation of why it's a red flag and what the PM should say about it.

### 2.6 Estimation Guidance

Provide context for story point discussion:
- **Repos touched:** list them
- **Greenfield vs modification:** is this new code or changing existing logic?
- **Test coverage:** do tests exist for the area being changed? Will new test infrastructure be needed?
- **Complexity signals:** number of services involved, external API integration, data migration, feature flags needed
- **Suggested t-shirt size:** S / M / L / XL with rationale

### 2.7 Acceptance Criteria Audit

Review each AC in the ticket:
- Flag vague ACs (e.g., "improve error handling" should be "return JSON with `error_code`, `error_message`, `is_retriable` fields")
- Flag untestable ACs (e.g., "better user experience" -- better how?)
- Flag missing ACs (e.g., no rollback criteria, no monitoring/alerting requirement)
- Suggest rewrites for any flagged AC

## Step 3: Generate Session Summary

After all individual briefs, produce a session overview:

### Suggested Agenda
Order tickets by:
1. Phase 0 quick wins first (builds momentum, shows progress)
2. Phase 1 foundational items (team understands the base layer)
3. Phase 2-3 dependent items (dependencies already discussed)

### Time Allocation
- **Quick (5 min):** Small scope, clear implementation, few questions expected
- **Standard (10 min):** Medium scope, some discussion points
- **Deep (15 min):** Large scope, architectural decisions, multiple alternatives

### Dependency Walkthrough Script
A 2-minute narrative the PM reads to explain the dependency map:
- Start with "Here's how these tickets connect..."
- Walk through the critical path
- Call out partner blockers explicitly
- End with "So the order we should build is..."

### Priority Cut List
If the session runs short, which tickets MUST be refined:
- List the 3-5 tickets that block the most other work
- For each, one sentence on why it can't wait

## TransferGo Context

- **Stack:** PHP (Laravel monolith + Symfony microservices), TypeScript (React admin frontend)
- **Repos:** `transfergo`, `payments-gateway-service`, `payments-settings-service`, `payments-settings-sdk` (PHP), `admin-frontend-old` (TS/React)
- **Jira project:** RL
- **RCA document:** `RCA_Report.md` in workspace root
- **Repo base URLs:**
  - Main backend: https://github.com/TransferGo/transfergo/blob/master/
  - Payments gateway: https://github.com/TransferGo/payments-gateway-service/blob/master/
  - Payments settings: https://github.com/TransferGo/payments-settings-service/blob/master/
  - Payments SDK: https://github.com/TransferGo/payments-settings-sdk/blob/master/
  - Admin frontend: https://github.com/TransferGo/admin-frontend-old/blob/master/

## Output Format

- Bullet points over paragraphs
- Bold section headers for scannability
- Questions in quotes so the PM can read them aloud
- Answers directly below each question (no separate section)
- Use tables for the session summary agenda
- Link to Jira tickets using `[RL-XXXX](https://transfergo.atlassian.net/browse/RL-XXXX)` format
- Link to GitHub files using full URLs

## Behavior Rules

- Adopt the CTO personality from the `cto-partner` skill: push back, be direct, highlight risks
- Never pad the brief with filler -- every line must earn its place
- If a ticket is missing critical sections (no User Story, no ACs, no Affected Files), say so bluntly and suggest what to add before refinement
- If a dependency looks artificial, call it out: "Engineers will challenge whether T10 really blocks T1 -- have a clear answer ready"
- Treat partner dependencies as the highest-risk items -- engineers hate being blocked by external parties
- When suggesting t-shirt sizes, err toward larger -- it's better to under-promise than to blow a sprint
- If the PM asks for a specific subset of tickets, only generate briefs for those -- don't pad with extras
