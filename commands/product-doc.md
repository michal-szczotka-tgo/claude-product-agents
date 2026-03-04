---
name: product-doc
description: Create high-quality product documents (PRDs, RCA reports, product concepts, technical specs) at executive review standard. Use when the user says "write a PRD", "create product doc", "draft a spec", "write documentation", "product concept", "RCA report", or needs to produce a polished product artifact.
---

# Product Documentation

Create product documents that meet the bar for review by a top Chief Product Officer.

## Quality Standard

Every document must be:
- **Data-backed**: Claims supported by metrics, research, or direct evidence
- **Concise**: No filler, no corporate fluff, every sentence earns its place
- **Structured**: Clear hierarchy, scannable headings, consistent formatting
- **Actionable**: Reader knows exactly what to do after reading
- **Honest**: Risks and tradeoffs stated plainly, not hidden

## Before Writing

1. Ask the user for the document type and purpose
2. Identify the audience (engineering, leadership, cross-functional)
3. Gather data: search codebase, query database, review existing docs
4. Outline the structure before writing full content

## Document Types

### PRD (Product Requirements Document)

Use the **TransferGo PRD Template** below for all PRDs. This is the default template for Step 3 of the CTO workflow.

### RCA (Root Cause Analysis)
- Incident timeline with facts only
- Impact quantification (users affected, revenue, duration)
- Root cause chain (not just symptoms)
- Contributing factors
- Remediation actions with owners and deadlines

### Product Concept
- Market opportunity with evidence
- User problem and current alternatives
- Proposed solution and differentiation
- Business model and unit economics
- Risks and open questions

### Technical Spec
- Context and motivation
- Proposed architecture (with diagrams if helpful)
- API contracts and data models
- Migration strategy
- Rollback plan

## TransferGo PRD Template

This is the standard PRD structure for TransferGo. Used in Step 3 of the CTO workflow (`cto-partner` skill) and whenever a PRD is requested.

### 1. Problem Statement
One paragraph. Data-backed. Answer: what is broken or missing, who is affected, and what is the impact?

Include supporting evidence: error rates, customer support volume, conversion drops, partner data, or Redshift queries. If data is unavailable, state what data is needed to validate.

### 2. Proposed Solution
Conceptual description of how we solve the problem. NOT implementation-level -- describe what changes from the user's perspective and what the system needs to do differently.

Keep this at the level a senior PM or engineering lead can review without reading code.

### 3. Success Metrics
Measurable, time-bound outcomes that define whether the solution worked.

Format:
| Metric | Current | Target | Measurement |
|--------|---------|--------|-------------|
| e.g., Card charge success rate | 87% | 93% | Redshift `charges` table, 30-day rolling |

### 4. UX Impact Summary
Filled in during Step 3a (UX Review) of the CTO workflow. If no UX review has been done yet, leave this as a placeholder.

Include:
- Screens affected (with Figma links if available)
- New UI states required (loading, error, empty, blocked, partial)
- UX gaps flagged
- Whether prototypes are needed
- Copy/messaging decisions that need PM input

### 5. Scope Boundaries

**In scope:**
- Bullet list of what this PRD covers

**Explicitly out of scope:**
- Bullet list of what this PRD does NOT cover and why

Being explicit about out-of-scope prevents scope creep during engineering refinement.

### 6. Technical Constraints & Architecture Boundaries
Which repos are involved, which domains own the logic, any hard constraints (e.g., "must use existing Temporal workflows", "cannot change the partner API contract").

Reference the Architecture Boundaries from the `cto-partner` skill for repo ownership.

### 7. Milestones / Phasing
High-level phases with estimated timelines. Each phase should be independently deployable where possible.

| Phase | Scope | Estimate | Dependencies |
|-------|-------|----------|--------------|
| 1 | e.g., Backend: persist decline state | M (1-3 days) | None |
| 2 | e.g., Monolith: map decline to user message | S (< 1 day) | Phase 1 |

### 8. Open Questions for Engineering Refinement
Bullet list of things that need engineer input before implementation can begin. These are intentionally left open -- the PRD defines the problem and direction, engineers refine the approach.

Examples:
- Should we use Redis TTL or a database column for the throttle state?
- What is the partner's actual retry window for soft declines?
- Can we reuse the existing rate limiting pattern from payouts?

## Formatting Rules

- Use markdown with clear heading hierarchy
- Tables for comparisons, timelines, metrics
- Bullet points over paragraphs
- Bold key terms and decisions
- Code blocks for technical details
- No emojis in formal documents
