---
name: lld-design-round-workflow
description: Orchestrates a full LLD design round — requirements, API, data model, flows, trade-offs — each followed by a Principal Engineer challenge and Staff Engineer response. Use when running a low-level design interview or producing a greenfield backend design doc.
---

**Produces**: A merged LLD document at `docs/design/problems/<problem-name>/lld.md` using the `docs/design/LLD-TEMPLATE.md` section structure, a per-stage debate log, and a consolidated confidence dashboard.

## Debate trigger rule

After each Staff sub-step (Na), evaluate the Staff output confidence before invoking the Principal review. This maps directly to the HITL rubric in `.cursor/CONFIDENCE-SCORING.md`:

| Staff output | HITL level | Action |
|-------------|-----------|--------|
| Confidence ≥ 90% AND Required = 0 AND Recommended = 0 | Optional only | **SKIP** Nb + Nc → AUTO-APPROVED → pass Staff output to next stage |
| Confidence 70–89% OR Recommended HITL > 0 | Recommended | **RUN** full debate (Nb + Nc) |
| Confidence < 70% OR Required HITL > 0 | Required | **RUN** full debate (Nb + Nc) — mandatory |

Log auto-approved stages as: `Stage N debate: AUTO-APPROVED (confidence NN% — Optional HITL only)`

## Pipeline

Each design stage runs as a confidence-gated debate. Pass the **revised** output (or Staff output if auto-approved) to the next stage.

```
For each stage N (1–5):

  Na. Staff specialist produces output
       │
       ├─ confidence ≥ 90%, Required HITL = 0, Recommended HITL = 0
       │     → AUTO-APPROVED — skip Nb + Nc
       │     → pass Staff output directly to stage N+1
       │
       └─ otherwise (confidence < 90% OR Required HITL > 0 OR Recommended HITL > 0)
             │
             Nb. lld-principal-reviewer challenges (≤ 5 force-ranked)
             │
             Nc. Staff specialist responds and revises
             │
             → pass revised output to stage N+1

After stage 5:
  ★  DEBATE GATE — all Blocking challenges resolved?
  6. lld-interviewer         (optional) Mock interview
  7. backend-design-validator (optional) Gap analysis vs existing code
```

## Stage invocation

| Stage | Sub-step | Agent file | Mode |
|-------|---------|-----------|------|
| 1 | Staff | `.cursor/agents/specialists/lld-requirements.md` | read-only |
| 1 | Review | `.cursor/agents/specialists/lld-principal-reviewer.md` | read-only |
| 1 | Respond | `.cursor/agents/specialists/lld-requirements.md` | read-only |
| 2 | Staff | `.cursor/agents/specialists/lld-api-designer.md` | read-only |
| 2 | Review | `.cursor/agents/specialists/lld-principal-reviewer.md` | read-only |
| 2 | Respond | `.cursor/agents/specialists/lld-api-designer.md` | read-only |
| 3 | Staff | `.cursor/agents/specialists/lld-data-modeler.md` | read-only |
| 3 | Review | `.cursor/agents/specialists/lld-principal-reviewer.md` | read-only |
| 3 | Respond | `.cursor/agents/specialists/lld-data-modeler.md` | read-only |
| 4 | Staff | `.cursor/agents/specialists/lld-sequence-flows.md` | read-only |
| 4 | Review | `.cursor/agents/specialists/lld-principal-reviewer.md` | read-only |
| 4 | Respond | `.cursor/agents/specialists/lld-sequence-flows.md` | read-only |
| 5 | Staff | `.cursor/agents/specialists/lld-trade-offs.md` | read-only |
| 5 | Review | `.cursor/agents/specialists/lld-principal-reviewer.md` | read-only |
| 5 | Respond | `.cursor/agents/specialists/lld-trade-offs.md` | read-only |
| 6 | — | `.cursor/agents/specialists/lld-interviewer.md` | interactive |
| 7 | — | `.cursor/agents/specialists/backend-design-validator.md` | read-only |

## Respond sub-step prompt pattern

```
Follow .cursor/agents/specialists/<specialist>.md.
Stage name: [stage]
Prior context: [all prior stage revised outputs]
Challenge report: [full output from lld-principal-reviewer for this stage]
Produce a revised stage output that addresses every Resolved or Partially resolved challenge.
Do not re-address Unresolved challenges — flag them for HITL instead.
```

## Gate rules

- **Debate gate** (after stage 5c): do not advance to stage 6 or mark design Approved if any stage has an Unresolved Blocking challenge.
- **Pipeline confidence** = minimum stage confidence across all Staff + Review sub-steps in stages 1–5.
- Do not advance to `backend-release-workflow` until debate gate passes and pipeline confidence ≥ 70%.

## Modes

| Mode | Stages | Use when |
|------|--------|----------|
| Full LLD with debate | 1a–5c | Greenfield design or written deliverable |
| Interview prep | 1a–5c + 6 | Mock practice round |
| Repo alignment | 1a–5c + 7 | Compare design to existing codebase |
| Post-design ship | 1a–5c approved → `backend-release-workflow` | Design approved, ready to implement |

## Output

```markdown
# LLD Design Round — Complete

## Problem
[stated problem from PROBLEM-BRIEF.md or user prompt]

## Confidence dashboard

| Stage | Sub-step | Agent | Confidence % | Debate triggered? | Blocking challenges | Verdict |
|-------|---------|-------|--------------|-----------------|---------------------|---------|
| 1 Requirements | Staff | lld-requirements | | YES / NO (AUTO-APPROVED) | — | — |
| 1 Requirements | Review | lld-principal-reviewer | | — | N | APPROVED / NEEDS REVISION |
| 1 Requirements | Respond | lld-requirements | | — | — | — |
| 2 API | Staff | lld-api-designer | | YES / NO | — | — |
| 2 API | Review | lld-principal-reviewer | | — | | |
| 2 API | Respond | lld-api-designer | | — | — | — |
| 3 Data model | Staff | lld-data-modeler | | YES / NO | — | — |
| 3 Data model | Review | lld-principal-reviewer | | — | | |
| 3 Data model | Respond | lld-data-modeler | | — | — | — |
| 4 Flows | Staff | lld-sequence-flows | | YES / NO | — | — |
| 4 Flows | Review | lld-principal-reviewer | | — | | |
| 4 Flows | Respond | lld-sequence-flows | | — | — | — |
| 5 Trade-offs | Staff | lld-trade-offs | | YES / NO | — | — |
| 5 Trade-offs | Review | lld-principal-reviewer | | — | | |
| 5 Trade-offs | Respond | lld-trade-offs | | — | — | — |
| 6 Interview | lld-interviewer | | — | — | — | — |
| 7 Code map | backend-design-validator | | — | — | — | — |

**Pipeline confidence**: NN%
**Debate gate**: PASSED / BLOCKED (N unresolved blocking challenges)

## Human review queue (consolidated)
- [ ] [Unresolved Blocking challenge — one validation question each, labeled by stage]

## Mock interview result (if run)
Score: / Recommendation: / Confidence: NN%

## Implementation gaps (if code mapped)
| LLD item | Code status | Gap |
|----------|-------------|-----|

## Recommended next step
[Resolve blocking challenges | Human sign-off on Required HITL | Run /backend-release | Revise section N]
```
