---
name: design-and-ship-workflow
description: Combines the full LLD design round and the backend release pipeline into one end-to-end workflow. Runs design stages 1–5, enforces an approval gate, then runs validate → test → perf → deploy. Use when building a new feature from scratch.
---

**Produces**: A completed LLD document at `docs/design/problems/<problem-name>/lld.md`, followed by a backend release pipeline report, with a single unified confidence dashboard spanning both phases.

## Pipeline

Run all stages in order. Do not skip the design gate before entering Phase 2.

```
╔══════════════════════════════╗
║  PHASE 1 — Design            ║
╠══════════════════════════════╣
║  1. lld-requirements         ║  FR/NFR, scope, assumptions
║         ↓                    ║
║  2. lld-api-designer         ║  REST contracts, error model, auth
║         ↓                    ║
║  3. lld-data-modeler         ║  Entities, indexes, storage choices
║         ↓                    ║
║  4. lld-sequence-flows       ║  Critical paths, failure branches
║         ↓                    ║
║  5. lld-trade-offs           ║  ADR-style decisions
╠══════════════════════════════╣
║  ★ DESIGN GATE (HARD)        ║  confidence ≥ 70% AND zero pending Required HITL
╠══════════════════════════════╣
║  PHASE 2 — Build & Ship      ║
╠══════════════════════════════╣
║  6. backend-design-validator ║  validate implementation vs LLD    (soft gate)
║         ↓                    ║
║  7. backend-test             ║  run and fix tests                 (HARD gate)
║         ↓ stop if FAIL       ║
║  8. backend-performance      ║  bottlenecks, hot paths            (soft gate)
║         ↓                    ║
║  9. backend-deploy           ║  build, ship, verify               (HARD gate on failure)
╚══════════════════════════════╝
```

## Stage invocation

| Stage | Agent file | Phase | Mode |
|-------|-----------|-------|------|
| 1 | `.cursor/agents/specialists/lld-requirements.md` | Design | read-only |
| 2 | `.cursor/agents/specialists/lld-api-designer.md` | Design | read-only |
| 3 | `.cursor/agents/specialists/lld-data-modeler.md` | Design | read-only |
| 4 | `.cursor/agents/specialists/lld-sequence-flows.md` | Design | read-only |
| 5 | `.cursor/agents/specialists/lld-trade-offs.md` | Design | read-only |
| 6 | `.cursor/agents/specialists/backend-design-validator.md` | Build | read-only |
| 7 | `.cursor/agents/specialists/backend-test.md` | Build | shell |
| 8 | `.cursor/agents/specialists/backend-performance.md` | Build | shell |
| 9 | `.cursor/agents/specialists/backend-deploy.md` | Build | shell |

Pass each stage's full output as context to the next stage. For stage 6, pass the complete LLD from stages 1–5 as the design reference.

## Gate rules

| Gate | Type | Condition |
|------|------|-----------|
| Design gate (after stage 5) | **Hard** | Phase 2 does not start if design confidence < 70% OR any Required HITL is pending. Stop and report the blocking items. |
| Test (stage 7) | **Hard** | FAIL or BLOCKED → do not run stages 8 or 9 |
| Deploy (stage 9) | **Hard** | FAILED → report exact rollback steps |
| Design validator (stage 6) | Soft | NEEDS CHANGES → warn user; log Critical items before stage 7 |
| Performance (stage 8) | Soft | Critical bottleneck → warn user before stage 9 |

## HITL rules

- Collect Overall confidence and Human review queue from every stage.
- Do not enter Phase 2 with pending Required HITL from Phase 1.
- Do not deploy with any pending Required HITL from Phase 2, unless the user explicitly overrides.
- **Pipeline confidence** = minimum stage confidence across all completed stages 1–9.
- Final verdict **READY TO SHIP** requires pipeline confidence ≥ 70% and zero pending Required HITL.

## Output

```markdown
# Design & Ship — Complete

## Problem
[stated problem]

## Phase 1 — Design confidence dashboard
| Stage | Agent | Confidence % | Required HITL | Pending HITL |
|-------|-------|--------------|---------------|--------------|
| 1 Requirements | lld-requirements | | | |
| 2 API | lld-api-designer | | | |
| 3 Data model | lld-data-modeler | | | |
| 4 Flows | lld-sequence-flows | | | |
| 5 Trade-offs | lld-trade-offs | | | |

**Design confidence**: NN% | **Design gate**: PASSED / BLOCKED

## Phase 2 — Build & ship confidence dashboard
| Stage | Agent | Verdict | Confidence % | Required HITL | Pending HITL |
|-------|-------|---------|--------------|---------------|--------------|
| 6 Design validate | backend-design-validator | APPROVED / WARN / BLOCK | | | |
| 7 Test | backend-test | PASS / FAIL | | | |
| 8 Performance | backend-performance | OK / WARN | | | |
| 9 Deploy | backend-deploy | SUCCESS / SKIPPED / FAILED | | | |

**Build confidence**: NN%

## Pipeline confidence (overall)
NN% (min across all 9 stages)

## Human review queue (consolidated)
- [ ] [Required item — one validation question each, labeled by stage]

## LLD output
`docs/design/problems/<problem-name>/lld.md`

## Overall verdict
READY TO SHIP | NOT READY | SHIPPED (local / staging / prod) | BLOCKED — design gate pending | BLOCKED — HITL pending

## Blockers
- [item — stage reference]

## Warnings
- [item — stage reference]

## Next steps
1.
```
