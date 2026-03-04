---
name: ux-review
description: Review UX impact of a feature or change by checking Figma for existing screens, flagging new UI states, and offering to create prototypes. Use when the user says "UX review", "check the UX", "what does the user see", "UX impact", or during Step 3a of the CTO workflow.
---

# UX Review

Review the user experience impact of a proposed feature or change. This skill runs after a PRD is drafted (Step 3 of the CTO workflow) and before technical exploration (Step 4).

## Why This Exists

Technical exploration should be informed by UX requirements. If the user will see a new error state, loading state, or blocked state, that affects which repos are in scope and what the API response needs to contain. UX review first, then explore.

## Inputs

You need one of:
- A feature/change description from the user
- A PRD (from the `product-doc` skill or Notion page)
- A Jira ticket draft

## Step 1: Check Existing Figma Designs

Use the Figma MCP to find screens related to the feature area.

**How:**
1. Ask the user for a Figma file URL if not already provided
2. Use `get_design_context` or `get_screenshot` to retrieve relevant screens
3. If no Figma URL is available, ask the user to describe or share the current UI

**What to look for:**
- Current screens the user sees for this flow
- Existing error states, empty states, loading states
- Design patterns already established (colors, spacing, component styles)
- Any design annotations or notes from designers

**If Figma MCP is unavailable:** Describe the expected screens in text based on your understanding of the feature, and ask the user to confirm or correct.

## Step 2: Analyze UX Impact

For the proposed change, answer these questions:

### What does the user see today?
- Which screens are involved in this flow?
- What happens on success? On failure?
- What information is displayed?

### What changes for the user?
- New screens or modified screens?
- New information displayed?
- Changed behavior or flow?

### What new states need handling?
Flag every new UI state the change introduces. Common ones:

- **Loading state** -- if a new async operation is added, what does the user see while waiting?
- **Error state** -- if a new failure mode exists, how is it communicated?
- **Empty state** -- if a list/view could now be empty, what does that look like?
- **Blocked state** -- if the user is temporarily prevented from an action (e.g., card throttled for 30 min), they need to see:
  - That the action is temporarily blocked
  - When they can retry (if applicable)
  - Alternative actions available (e.g., use a different card)
- **Partial state** -- if an operation partially succeeds, what does the user see?
- **Permission/eligibility state** -- if the feature depends on user eligibility, what does a non-eligible user see?

## Step 3: Flag Gaps

Summarize UX gaps in a clear list:

```
UX GAPS:
1. [GAP] No error state designed for card throttling -- user needs to see: blocked reason, retry time, alternative payment option
2. [GAP] Loading state missing for async partner verification -- current flow assumes sync response
3. [OK] Success flow is covered by existing design
4. [QUESTION] Is the retry timer shown as a countdown or a static timestamp?
```

## Step 4: Ask the PM

Present your findings and ask:

1. **"Should I create Figma prototypes for these new states?"**
   - If yes: trigger the `/figma-prototype` skill, passing:
     - UX spec file path (output of Step 3, or ask PM for the path)
     - Figma reference file key (ask PM if not already provided)
     - Output Figma file URL
     - Feature slug for naming
   - Do NOT attempt to create Figma nodes directly via the Figma MCP — always delegate to `/figma-prototype`
   - If PM wants text description only (no plugin): describe the screens in detail (layout, copy, components) for a designer to build
   - If no: document the gaps in the PRD for the design team to address separately

2. **"Are there any states I missed?"** -- the PM may know about edge cases from customer support data or partner behavior

## Step 5: Verify Spec Against Prototype

After `/figma-prototype` has run and the plugin has been executed in Figma:

1. Ask the user to share the output Figma file URL (or confirm it's the same file already provided)
2. Use Figma MCP `get_design_context` or `get_metadata` on the output file to retrieve the created frame names
3. Resolve all `[CONFIRM]` annotations in the UX spec file:
   - Replace with verified component or frame names from the live Figma file
   - If still unconfirmable, replace with `[UNRESOLVED: reason]`
4. Update the Notion PRD's "UX Impact Summary" section with a link to the prototype

**If prototype was skipped** (PM said no): skip this step and proceed to Step 6.

## Step 6: Update the PRD

Add a **UX Impact Summary** section to the PRD with:

- Screens affected (with Figma links if available)
- New states identified (loading, error, empty, blocked, partial)
- Gaps flagged
- Whether prototypes are needed
- Any copy/messaging decisions that need PM input

## Output Format

```
## UX Impact Summary

### Screens Affected
- [Screen name] -- [what changes] -- [Figma link if available]

### New States Required
- [State type]: [description of what user needs to see]

### Gaps
- [GAP/OK/QUESTION]: [description]

### Prototype Needed?
- [Yes/No] -- [list of screens if yes]

### Open UX Questions
- [Question for PM or design team]
```

## Integration with CTO Workflow

This skill is called at **Step 3a** of the CTO workflow (`cto-partner` skill):
- **Input**: PRD drafted in Step 3
- **Output**: UX Impact Summary added to the PRD
- **Next step**: Step 4 (Exploration), now informed by UX requirements
