---
name: slack-to-jira
description: Convert a flagged Slack message into a structured Jira RL ticket fast. Use when the user says "slack to jira", "triage this", "ticket this message", "create ticket from slack", pastes a Slack message, shares a Slack URL, or wants to convert a Slack conversation into a trackable work item.
---

# Slack-to-Jira Triage

Convert a Slack message into a structured RL Jira ticket in under 2 minutes. The user flags the message; you classify, enrich, draft, and create.

## Input Modes

Accept input in any of these forms (try in order):

1. **Slack URL** -- If the Slack MCP is authenticated, fetch the message and its thread.
2. **Pasted message text** -- User copies the message directly into chat.
3. **Channel scan** -- User says "check #channel-name for recent messages". Use Slack MCP `search` to find recent actionable messages and present a numbered list for selection.

If the Slack MCP is unavailable or errored, skip to pasted text and tell the user: "Slack MCP isn't connected -- paste the message text and I'll work with that."

## Step 1: Classify

Read the message and determine:

| Field | How to Decide |
|---|---|
| **Type** | Bug (something broken), Feature (something new), Improvement (something better), Incident (production issue), Task (action item) |
| **Priority** | High if: "production", "blocking", "customers affected", "urgent", "P1". Normal otherwise. |
| **Domain** | Match keywords to TransferGo domains: Payments/Charges/Payouts, Compliance/KYC/AML, Risk/Fraud, Onboarding, Profiles, Pricing. Default to the most specific match. |

Present classification to user for confirmation: "I'm reading this as a **[Type]** in **[Domain]**, priority **[Priority]**. Correct?"

## Step 2: Clarify (0-2 questions max)

Ask ONLY if the message is genuinely ambiguous. Skip this step if you can construct a reasonable ticket.

Acceptable questions:
- "Who is affected -- all users or a specific corridor/partner?"
- "What's the desired outcome here?"
- "Is this already tracked somewhere?"

Do NOT ask a checklist of questions. Be surgical. Goal: under 30 seconds of user time.

## Step 3: Duplicate Check

Before drafting, search Jira for potential duplicates:

```
JQL: project = RL AND summary ~ "[key terms from message]" AND status != Done ORDER BY created DESC
Limit: 5
```

If potential duplicates exist, show them: "Found similar tickets: RL-XXX, RL-YYY. Should I still create a new one, or add to an existing ticket?"

## Step 4: Optional Codebase Exploration

Trigger this step ONLY if the message references specific technical areas (partner names, service names, error codes, endpoints). Skip for vague or business-only messages.

When triggered, do a quick search (not a full exploration):
- Grep across workspace repos for the mentioned keyword (partner name, error code, endpoint)
- Identify which repos and directories are affected
- Note any existing patterns relevant to the fix

This feeds the **Affected Areas** section of the ticket. Keep it fast -- 30 seconds max.

## Step 5: Draft the Ticket

Follow the `create-jira-issue` skill structure exactly. The ticket body uses this format:

### Title Convention

`[Domain] Action-oriented summary`

Examples:
- `[Charges] Investigate elevated decline rates on Nuvei card payments`
- `[Compliance] Add screening for new sanction list entries`
- `[Payouts] Fix timeout errors for SWIFT payouts to Nigeria`

### Ticket Body Sections (in order)

**1. TL;DR** (required, always first)
Two sentences. First: the problem. Second: the direction.

**2. Current State vs Desired State** (required)
Bullet points extracted from the Slack message. Apply confidence markers:
- `[VERIFIED]` -- confirmed by reading actual code
- `[UNVERIFIED -- needs X]` -- not verified
- `[FROM SLACK]` -- directly quoted from the Slack message (use this liberally)

**3. Acceptance Criteria** (required)
Testable checkboxes. Include at least one edge case.

**4. Conceptual Approach** (required)
High-level direction. NO file paths, NO class names, NO pseudo-code.

**5. Affected Areas** (required)
Repos and domains. Use GitHub directory links:
- `[repo]` `payments-gateway-service` -- [source/src/Charge/](https://github.com/TransferGo/payments-gateway-service/tree/master/source/src/Charge/) -- charge workflow
- `[repo]` `transfergo` -- [source/TransferGo/Payments/](https://github.com/TransferGo/transfergo/tree/master/source/TransferGo/Payments/) -- payment orchestration

If Step 4 was skipped, write: "Affected areas need engineer triage during refinement."

**6. Dependencies** (required)
If mentioned in thread or obvious. Otherwise: "None identified from Slack context."

**7. Risks & Notes** (if applicable)
Include rollback plan for risky changes.

**8. Source** (always include)
Link to the original Slack message (URL if available) or note "Reported via Slack by [author] on [date]".

### Additional Fields
- **Labels**: `from-slack` (always add this label for traceability)
- **Priority**: As classified in Step 1

## Step 6: User Approval

Present the full draft to the user. Wait for explicit confirmation before creating.

Acceptable responses:
- "go" / "create" / "ship it" -- create the ticket
- Any modification request -- update the draft and re-present
- "skip" / "cancel" -- abort

NEVER auto-create without approval.

## Step 7: Create in Jira

Create the ticket via Jira MCP with:
- `project_key: RL`
- `issue_type: [Type from Step 1]`
- `labels: ["from-slack"]`
- `priority: [Priority from Step 1]`

If Step 4 was run, post a follow-up comment with technical context (file paths, existing patterns, related code).

Return: "Created **RL-XXX**: [title]. [Link to ticket]"

## Batch Mode

When the user says "scan #channel" or "triage recent messages":

1. Fetch recent messages from the channel via Slack MCP search
2. Filter out noise (greetings, reactions-only, bot messages)
3. Present a numbered list of actionable messages with one-line summaries
4. User selects which ones to convert
5. Run Steps 1-7 for each selected message sequentially
6. At the end, summarize: "Created X tickets: RL-XXX, RL-YYY, RL-ZZZ"

## Behavior Rules

- **Speed over perfection**: A good-enough ticket now beats a perfect ticket in 15 minutes.
- **Mark uncertainty**: Use `[UNVERIFIED]` and `[FROM SLACK]` liberally. Never fabricate technical details.
- **Preserve voice**: Quote the original Slack message in the TL;DR or Current State.
- **One ticket per problem**: If a Slack message contains multiple issues, ask "This mentions X and Y -- create separate tickets?"
- **from-slack label**: Always apply for traceability and filtering.
- **Minimal questions**: Every question costs 30+ seconds. Only ask when you genuinely cannot proceed.

## TransferGo Domain Keyword Map

| Keywords | Domain |
|---|---|
| charge, pay-in, card payment, 3DS, decline, checkout | Charges |
| payout, bank transfer, SWIFT, SEPA, withdrawal | Payouts |
| KYC, verification, document, identity, selfie | KYC |
| AML, sanction, screening, PEP, watchlist | Compliance |
| risk, fraud, suspicious, chargeback, dispute | Risk |
| exchange, FX, rate, conversion, currency | CurrencyExchange |
| IBAN, bank details, BIC, sort code, routing | BankDetails |
| onboarding, registration, signup, welcome | Onboarding |
| profile, account, user settings, preferences | Profiles |
| pricing, fee, markup, corridor pricing | Pricing |
| admin, backoffice, internal tool, ops | Admin |

## TransferGo Repo Links

- Main backend: https://github.com/TransferGo/transfergo/tree/master/
- Payments gateway: https://github.com/TransferGo/payments-gateway-service/tree/master/
- Payments settings: https://github.com/TransferGo/payments-settings-service/tree/master/
- Payments SDK: https://github.com/TransferGo/payments-settings-sdk/tree/master/
- Bank details: https://github.com/TransferGo/bank-details-lookup-service/tree/master/
- Admin frontend: https://github.com/TransferGo/admin-frontend-old/tree/master/
