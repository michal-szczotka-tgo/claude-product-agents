# Workflow Diagram

Full pipeline from idea to shipped code, with every agent/skill mapped to its step.

```mermaid
flowchart TD
    %% ── Entry Points ──────────────────────────────────────────
    IDEA([💡 Product Idea / Bug Report])
    SLACK([💬 Slack Message])
    TICKET([🎫 Existing Jira Ticket])

    %% ── Discovery ─────────────────────────────────────────────
    subgraph DISCOVERY ["🔍 Discovery  ·  orchestrated by /cto-partner"]
        direction TB
        D1["**Step 1–2**\n/cto-partner\nBrainstorm & clarify until scope is clear"]
        D2["**Step 3**\n/product-doc\nCreate PRD on Notion"]
        D3["**Step 3a**\n/ux-review\nUX impact analysis · flag gaps"]
        D4["**Step 4 (if approved)**\n/figma-prototype\nGenerate Figma plugin from UX spec"]
        D5["**Step 5**\nVerify spec vs prototype\nResolve all ﹝CONFIRM﹞ annotations"]
        D6["**Step 4**\n/explore\nCodebase analysis across all repos"]
        D7["/db-explore\nRedshift schema (optional)"]
        D8["**Step 4h**\nSurface considerations & open questions\n→ feeds PRD Section 8 + ticket body"]
    end

    %% ── Planning ──────────────────────────────────────────────
    subgraph PLANNING ["📋 Planning & Tickets"]
        direction TB
        P1["**Step 5**\n/create-plan\nPhased implementation plan"]
        P2["**Step 6**\n/create-jira-issue\nCreate Jira RL tickets"]
        P3["/refinement-prep\nData-backed refinement talking points"]
        P4["/handoff-to-implementation\nValidate PRD · structured engineering handoff"]
    end

    %% ── Execution ─────────────────────────────────────────────
    subgraph EXECUTION ["⚙️ Execution & Review"]
        direction TB
        E1["/execute\nImplement from plan · track progress"]
        E2["/jira-to-code\nTicket → explore → implement → PR"]
        E3["/create-pr\nStructured GitHub PR"]
        E4["/review\nCode review · PHP / Symfony / Laravel / TS"]
        E5["/peer-review\nVerify external review findings"]
        E6["/post-refinement\nCondense ticket post-refinement"]
    end

    %% ── Quick Paths ───────────────────────────────────────────
    subgraph QUICK ["⚡ Quick Paths (bypass full discovery)"]
        direction TB
        Q1["/slack-to-jira\nSlack message → Jira ticket"]
        Q2["/create-jira-issue\nDirect bug / feature ticket"]
    end

    %% ── Utilities ─────────────────────────────────────────────
    subgraph UTILS ["🛠 Utilities"]
        direction TB
        U1["/learning-opportunity\nExplain concept at 3 depth levels"]
        U2["/document\nUpdate CHANGELOG + docs after ship"]
        U3["/keybindings-help\nCustomize Claude Code shortcuts"]
    end

    %% ── Flow ──────────────────────────────────────────────────

    %% Entry → Discovery
    IDEA --> D1

    %% Discovery internal flow
    D1 --> D2
    D2 --> D3
    D3 -- "prototypes\napproved" --> D4
    D4 --> D5
    D3 --> D6
    D6 -. "schema\nneeded" .-> D7
    D6 --> D8

    %% Discovery → Planning
    D8 --> P1
    P1 --> P2
    P2 --> P3
    P3 --> P4

    %% Planning → Execution
    P4 --> E1
    E1 --> E3
    E3 --> E4
    E4 -. "external\nfeedback" .-> E5
    E4 --> E6

    %% Quick paths
    SLACK --> Q1 --> P2
    TICKET --> E2 --> E3

    %% Utilities (standalone)
    IDEA -.-> Q2 --> P2

    %% Style
    classDef entry fill:#e8f4fd,stroke:#1a73e8,color:#1a1a1a
    classDef skill fill:#f8f9fb,stroke:#6c727a,color:#212121
    classDef quick fill:#fef9e7,stroke:#f59e0b,color:#1a1a1a
    classDef util fill:#f0fdf4,stroke:#16a34a,color:#1a1a1a

    class IDEA,SLACK,TICKET entry
    class Q1,Q2 quick
    class U1,U2,U3 util
```

---

## Stage Summary

| Stage | Skills | Entry | Output |
|---|---|---|---|
| Discovery | `/cto-partner` · `/product-doc` · `/ux-review` · `/figma-prototype` · `/explore` · `/db-explore` | Raw idea or bug report | PRD on Notion + UX spec + prototype + open questions table |
| Planning | `/create-plan` · `/create-jira-issue` · `/refinement-prep` · `/handoff-to-implementation` | Exploration output | Phased plan + Jira tickets + engineering handoff |
| Execution | `/execute` · `/jira-to-code` · `/create-pr` · `/review` · `/peer-review` · `/post-refinement` | Jira ticket or plan | Merged PR + clean ticket |
| Quick paths | `/slack-to-jira` · `/create-jira-issue` | Slack message or known bug | Jira ticket (no full discovery) |
| Utilities | `/learning-opportunity` · `/document` · `/keybindings-help` | Any point in workflow | Teaching, docs update, config |

---

## Skill Invocation Map

Who calls whom (explicit chaining between skills):

```
/cto-partner
  ├── calls /product-doc        (Step 3)
  ├── calls /ux-review          (Step 3a)
  │     └── calls /figma-prototype  (Step 4, if approved)
  └── calls /explore            (Step 4)

/jira-to-code
  └── calls /create-pr          (internally)

/ux-review
  └── calls /figma-prototype    (Step 4, if approved)
```

All other skills are invoked directly by the user.
