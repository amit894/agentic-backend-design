# Pipeline Diagram

Visual reference for the backend-lld-kit workflow system.

---

## 1 — Start here: fill PROBLEM-BRIEF.md, then pick a command

```mermaid
flowchart TD
    PB["📋 PROBLEM-BRIEF.md
    ─────────────────────────
    Title
    Output folder
    One-line summary
    Core scenarios
    Constraints · Scale · Latency
    Integrations
    Non-goals
    Open questions
    Interview context"]

    PB --> INTENT{"Which command?"}

    INTENT -->|"Design only\n/lld-round"| LLD
    INTENT -->|"Build & ship only\n/backend-release"| REL
    INTENT -->|"Design + build + ship\n/design-and-ship"| DS

    subgraph LLD ["🎨  /lld-round — LLD Design Round"]
        direction TB
        L1["5 design stages
        each runs the 3-step debate loop ↓"]
        L2["★ Design Gate
        confidence ≥ 70%
        zero Unresolved Blocking challenges
        zero pending Required HITL"]
        L3["lld-interviewer  (optional)
        backend-design-validator  (optional)"]
        L1 --> L2 --> L3
    end

    subgraph REL ["🚀  /backend-release — Release Pipeline"]
        direction TB
        R1["backend-design-validator  (soft gate)"]
        R2["backend-test  ── HARD gate"]
        R3["backend-performance  (soft gate)"]
        R4["backend-deploy  ── HARD gate"]
        R1 --> R2 --> R3 --> R4
    end

    subgraph DS ["⚡  /design-and-ship — End to End"]
        direction TB
        D1["Phase 1 · LLD + Debate
        same 5 stages as /lld-round"]
        D2["★ Design Gate  ── HARD"]
        D3["Phase 2 · Build & Ship
        same 4 stages as /backend-release"]
        D1 --> D2 --> D3
    end

    LLD -->|"writes"| OUT["📄 docs/design/problems/
    name/lld.md"]
    DS  -->|"writes"| OUT
    REL -->|"produces"| RPT["📊 Release Report
    READY TO SHIP / BLOCKED"]
    DS  -->|"produces"| RPT
```

---

## 2 — The per-stage debate loop (confidence-gated, runs × 5)

The debate is **not unconditional**. It is triggered by the Staff Engineer's confidence score and HITL counts, using the `.cursor/CONFIDENCE-SCORING.md` rubric.

```mermaid
flowchart TD
    A["🧑‍💻  Staff Engineer  —  Na
    ──────────────────────────────
    lld-requirements · lld-api-designer
    lld-data-modeler · lld-sequence-flows
    lld-trade-offs
    ──────────────────────────────
    Produces stage output
    with confidence % and HITL counts"]

    A --> GATE{"Debate trigger check"}

    GATE -->|"confidence ≥ 90%
    Required HITL = 0
    Recommended HITL = 0"| AUTO["✅  AUTO-APPROVED
    Skip Nb + Nc
    Pass Staff output
    directly to Stage N+1"]

    GATE -->|"confidence < 90%
    OR Required HITL > 0
    OR Recommended HITL > 0"| B

    B["🔍  Principal Engineer  —  Nb
    ──────────────────────────────
    lld-principal-reviewer
    ──────────────────────────────
    Force-ranks ≤ 5 challenges
    Rates: Blocking / Non-blocking
    One concrete question each"]

    B --> C["🧑‍💻  Staff Engineer  —  Nc
    ──────────────────────────────
    same specialist
    ──────────────────────────────
    Responds to each challenge
    Produces revised output"]

    B -->|"Unresolved\nBlocking"| H["🔴  Required HITL
    Blocks Design Gate"]

    C -->|"revised output\nas context"| NEXT["Stage N+1"]
    AUTO --> NEXT

    style AUTO fill:#22aa44,color:#fff
    style H fill:#cc2222,color:#fff
```

**Trigger thresholds from CONFIDENCE-SCORING.md:**

| Confidence | HITL level | Debate |
|-----------|-----------|--------|
| ≥ 90% · Verified · Optional HITL only | Low risk | AUTO-APPROVED — skip |
| 70–89% · Inferred · Recommended HITL | Medium risk | RUN debate |
| < 70% · Assumed · Required HITL | High risk | RUN debate (mandatory) |

---

## 3 — Confidence & gate rules at a glance

```mermaid
flowchart TD
    CONF["Confidence scoring — every claim in every report
    ───────────────────────────────────────────────────
    90–100%  High    · Verified in code, test output, or docs
    70–89%   Medium  · Inferred from structure or config
    50–69%   Low     · Assumed; stated explicitly
    < 50%    Very low · Speculative; missing data"]

    CONF --> HITL["HITL classification
    ─────────────────────────────────────────────
    Required      confidence < 70%  OR  Assumed on Must item
    Recommended   70–89%  OR  Inferred on security/scale item
    Optional      ≥ 90%  AND  Verified"]

    HITL --> GATE["Design Gate passes when ALL THREE hold
    ────────────────────────────────────────────────────────
    ✅  Pipeline confidence ≥ 70%  (min across all sub-steps)
    ✅  Zero Unresolved Blocking challenges  (from debate rounds)
    ✅  Zero pending Required HITL items"]

    GATE -->|"PASSED"| SHIP["Phase 2 · Build & Ship
    or mark design Approved"]
    GATE -->|"BLOCKED"| RESOLVE["Resolve items →
    re-run affected stage"]
```

---

## 4 — Full artifact map

```mermaid
flowchart LR
    CMD[".cursor/commands/
    ──────────────────
    lld-round.md
    backend-release.md
    design-and-ship.md
    extend-workflow.md"]

    SKL[".cursor/skills/
    ──────────────────
    lld-design-round/
    backend-release-pipeline/
    design-and-ship/"]

    WFL[".cursor/workflows/
    ──────────────────
    lld-design-round-workflow.md
    backend-release-workflow.md
    design-and-ship-workflow.md"]

    AGT[".cursor/agents/specialists/
    ──────────────────────────────
    lld-requirements.md
    lld-api-designer.md
    lld-data-modeler.md
    lld-sequence-flows.md
    lld-trade-offs.md
    lld-principal-reviewer.md  ← debate
    lld-interviewer.md
    backend-design-validator.md
    backend-test.md
    backend-performance.md
    backend-deploy.md"]

    RUB[".cursor/
    CONFIDENCE-SCORING.md
    WORKFLOW-ARCHITECTURE.md
    PIPELINE-DIAGRAM.md  ← this file"]

    CMD -->|"invokes"| WFL
    SKL -->|"routes to"| WFL
    WFL -->|"delegates to"| AGT
    RUB -->|"governs"| AGT
    RUB -->|"governs"| WFL
```
