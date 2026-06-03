---
name: lld-design-round-workflow
description: Orchestrates a full LLD design round — requirements, API, data model, flows, trade-offs, optional mock interview, optional code gap analysis. Use when running a low-level design interview or producing a greenfield backend design doc.
---

**Produces**: A merged LLD document at `docs/design/problems/<problem-name>/lld.md` (create the folder if it does not exist) using the `docs/design/LLD-TEMPLATE.md` section structure, a confidence dashboard, and a consolidated human review queue.

## Pipeline

Run stages in order. Pass each stage's full output as context to the next stage.

```
1. lld-requirements      FR/NFR, scope, assumptions
        ↓
2. lld-api-designer      REST contracts, error model, auth
        ↓
3. lld-data-modeler      Entities, indexes, storage choices
        ↓
4. lld-sequence-flows    Critical paths, failure branches
        ↓
5. lld-trade-offs        ADR-style decisions with rejected alternatives
        ↓
6. lld-interviewer       (optional) Mock interview — stress-test weak areas
        ↓
7. backend-design-validator  (optional) Gap analysis vs existing code
```

## Stage invocation

| Stage | Agent file | Mode |
|-------|-----------|------|
| 1 | `.cursor/agents/specialists/lld-requirements.md` | read-only |
| 2 | `.cursor/agents/specialists/lld-api-designer.md` | read-only |
| 3 | `.cursor/agents/specialists/lld-data-modeler.md` | read-only |
| 4 | `.cursor/agents/specialists/lld-sequence-flows.md` | read-only |
| 5 | `.cursor/agents/specialists/lld-trade-offs.md` | read-only |
| 6 | `.cursor/agents/specialists/lld-interviewer.md` | interactive |
| 7 | `.cursor/agents/specialists/backend-design-validator.md` | read-only |

## Gate rules

- Do not mark design **Approved** if any stage 1–5 has a pending **Required** HITL item.
- **Pipeline confidence** = minimum stage confidence across stages 1–5 (and stage 7 if run).
- Do not advance to `backend-release-workflow` until pipeline confidence ≥ 70% and zero pending Required HITL.

## Modes

| Mode | Stages | Use when |
|------|--------|----------|
| Full LLD doc | 1–5 | Greenfield design or written deliverable |
| Interview prep | 1–5 + 6 | Mock practice round |
| Repo alignment | 1–5 + 7 | Compare design to existing codebase |
| Post-design ship | 1–5 approved → `backend-release-workflow` | Design approved, ready to implement |

## Output

```markdown
# LLD Design Round — Complete

## Problem
[stated problem from PROBLEM-BRIEF.md or user prompt]

## Confidence dashboard
| Stage | Agent | Confidence % | Required HITL | Pending HITL |
|-------|-------|--------------|---------------|--------------|
| 1 Requirements | lld-requirements | | | |
| 2 API | lld-api-designer | | | |
| 3 Data model | lld-data-modeler | | | |
| 4 Flows | lld-sequence-flows | | | |
| 5 Trade-offs | lld-trade-offs | | | |
| 6 Interview | lld-interviewer | | | |
| 7 Code map | backend-design-validator | | | |

**Pipeline confidence**: NN%

## Human review queue (consolidated)
- [ ] [Required item — one validation question each]

## Mock interview result (if run)
Score: / Recommendation: / Interviewer confidence: NN%

## Implementation gaps (if code mapped)
| LLD item | Code status | Gap |
|----------|-------------|-----|

## Recommended next step
[Human sign-off on Required HITL | Implement | Run /backend-release | Revise section N]
```
