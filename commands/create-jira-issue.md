---
name: create-jira-issue
description: Capture bugs, features, and improvements as structured Jira issues with conceptual approaches, GitHub-linked affected areas, and dependency tracking. Use when the user says "create issue", "log a bug", "new ticket", "track this", "file a feature request", or wants to capture work items.
---

# Create Issue

User is mid-development and thought of a bug/feature/improvement. Capture it fast so they can keep working.

## Title Convention

`[Domain] Action-oriented summary`

Examples:
- `[Charges] Classify gateway declines as terminal/retriable/user-action`
- `[Payouts] Add rate limiting for Nuvei corridor`
- `[KYC] Surface document rejection reason to the user`

Always mention the business domain. Never use jargon-only titles.

## Ticket Structure

Every ticket follows this format (sections in order). The ticket MUST clearly separate the PROBLEM (sections 1-3) from the DIRECTION (sections 4-6). Engineers own the implementation -- the ticket defines the problem and conceptual direction.

### 1. TL;DR (required, ALWAYS FIRST)

Two sentences maximum. First sentence: what's the problem. Second sentence: how we want to solve it. The reader understands the ticket in 10 seconds.

### 2. Current State vs Desired State (required)

Bullet points. Be specific with numbers, error codes, and data.

**Confidence markers** -- apply to every factual claim:
- `[VERIFIED]` -- confirmed by reading actual code (cite file path)
- `[UNVERIFIED -- needs X]` -- not verified, state what's needed to verify (e.g., "needs Redshift query", "needs CKO docs review")
- `[FROM PARTNER DOCS]` -- from external documentation (link required)

Never present unverified numbers as facts. If acceptance rate data, error counts, or partner behavior hasn't been confirmed, say so explicitly.

### 3. Acceptance Criteria (required)

Checkboxes. Testable conditions that define "done".

Include edge cases -- ask "what happens when the unexpected occurs?" Examples:
- What happens with UNKNOWN or UNMAPPED states?
- What if the partner returns an undocumented error code?
- What if the service is unavailable?
- What about concurrent requests for the same resource?

### 4. Conceptual Approach (required)

High-level description of WHAT needs to happen, not HOW to code it.

**Think about:**
- Which services are involved and what role each plays
- What data flows between services
- What the event/lifecycle looks like
- Which partners are affected
- What the user should experience

**Rules:**
- NO exact file paths in this section
- NO pseudo-code or class names
- NO code-level prescriptions
- DO describe service responsibilities and data flows
- DO describe user-visible behavior changes

**Example:**
> PGW needs to persist card-level failure state with a TTL. When a new charge arrives for the same card, check this state before initiating the partner call. If blocked, return a response the monolith can translate into user-facing messaging.

### 5. Affected Areas (required)

Which repos, which domains/modules, which partner integrations. Link to GitHub directories, not individual files.

Format:
- `[repo]` `payments-gateway-service` -- [source/src/Charge/](https://github.com/TransferGo/payments-gateway-service/tree/master/source/src/Charge/) -- charge workflow and partner integration
- `[repo]` `transfergo` -- [source/TransferGo/Payments/](https://github.com/TransferGo/transfergo/tree/master/source/TransferGo/Payments/) -- charge status mapping and client response

### 6. Dependencies (required if any)

- **Depends on:** [TICKET-ID] -- why this must ship first
- **Blocks:** [TICKET-ID] -- what can't start until this ships
- **Frontend dependency:** describe what the frontend needs from this backend change (API response shape, new fields, error codes)
- **Backend dependency:** describe what the backend must provide for this frontend change

If truly independent, write "None".

### 7. Risks & Notes (if applicable)

Bullet points. Include rollback plan for risky changes. Flag any business risks (e.g., liability shifts, chargeback exposure).

## What Goes in Jira Comments (NOT the ticket body)

The following context is valuable for engineers during refinement but should NOT clutter the ticket body. Post it as a Jira comment after ticket creation:

- **Existing Infrastructure**: File paths, class names, existing endpoints/services, reusable patterns, exact code references. Format:
  - `[endpoint]` `GET /api/payments/charge/{id}/status` in `transfergo` monolith -- [GetPartnerChargeStatusController.php](https://github.com/TransferGo/transfergo/blob/master/source/app/controllers/Api/PaymentProcessing/PartnerPayment/GetPartnerChargeStatusController.php)
  - `[pattern]` Payout rate limiting in PGW -- `source/src/Payment/Infrastructure/Partners/Base/Activity/RateLimit/`

## TransferGo Repo Links

When referencing repos or directories, use the correct base URL:
- Main backend: https://github.com/TransferGo/transfergo/blob/master/
- Payments gateway: https://github.com/TransferGo/payments-gateway-service/blob/master/
- Payments settings: https://github.com/TransferGo/payments-settings-service/blob/master/
- Payments SDK: https://github.com/TransferGo/payments-settings-sdk/blob/master/
- Admin frontend: https://github.com/TransferGo/admin-frontend-old/blob/master/

Source code lives under `source/` for PHP repos and `apps/` for the frontend.

## How to Get There

**Ask questions** to fill gaps -- concise, respect the user's time. Usually need:
- What's the issue/feature
- Current behavior vs desired behavior
- Type (bug/feature/improvement) and priority if not obvious

**Search for context** -- this is MANDATORY, not optional:
- Grep ALL repos in the workspace to find relevant code and verify claims
- Check for existing implementations BEFORE proposing new ones
- Trace the request flow end-to-end to understand repo ownership
- Check AGENTS.md/CLAUDE.md in the repo for conventions
- Note risks or cross-repo dependencies

**Think holistically** -- every backend ticket should consider:
- Does the frontend need to consume a new API field?
- Does the error response format change?
- Are there other services that depend on this behavior?
- Which repo actually owns this endpoint/workflow?

## Behavior Rules

- Be conversational -- ask what makes sense, not a checklist
- Default priority: normal, effort: medium (ask only if unclear)
- Bullet points over paragraphs
- Always include the frontend dependency note in the Dependencies section on backend tickets
- NEVER fabricate technical details. If you haven't found it in the code, don't claim it exists.
- Keep the ticket body clean and readable. Technical deep-dives belong in comments.
