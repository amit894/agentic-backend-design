---
name: design-and-ship-workflow
description: Combines the full LLD design round (with per-stage Principal Engineer debate) and the backend release pipeline into one end-to-end workflow. Use when building a new feature from scratch.
---

**Produces**: A completed LLD document at `docs/design/problems/<problem-name>/lld.md`, a per-stage debate log, and a backend release pipeline report — with a single unified confidence dashboard spanning both phases.

## Debate trigger rule (Phase 1)

After each Staff sub-step (Na), check confidence before invoking the Principal review. Mirrors `.cursor/CONFIDENCE-SCORING.md` HITL thresholds:

| Staff output | Action |
|-------------|--------|
| Confidence ≥ 90% AND Required HITL = 0 AND Recommended HITL = 0 | AUTO-APPROVED — skip Nb + Nc |
| Confidence 70–89% OR Recommended HITL > 0 | RUN debate (Nb + Nc) |
| Confidence < 70% OR Required HITL > 0 | RUN debate (Nb + Nc) — mandatory |

## Pipeline

```
╔══════════════════════════════════════════════════════╗
║  PHASE 1 — Design (confidence-gated debate)          ║
╠══════════════════════════════════════════════════════╣
║  For each stage N (1–5):                             ║
║                                                      ║
║  Na. Staff specialist produces output                ║
║       │                                              ║
║       ├─ conf ≥ 90%, Req HITL = 0, Rec HITL = 0     ║
║       │      → AUTO-APPROVED (skip Nb + Nc)          ║
║       │                                              ║
║       └─ otherwise                                   ║
║              Nb. lld-principal-reviewer challenges   ║
║              Nc. Staff specialist responds + revises  ║
║                                                      ║
║  Stages: Requirements · API · Data Model ·           ║
║          Flows · Trade-offs                          ║
╠══════════════════════════════════════════════════════╣
║  ★ DESIGN GATE (HARD)                                ║
║  confidence ≥ 70%                                    ║
║  zero Unresolved Blocking challenges                  ║
║  zero pending Required HITL                          ║
╠══════════════════════════════════════════════════════╣
║  PHASE 2 — Build & Ship                  ║
╠══════════════════════════════════════════╣
║  6. backend-design-validator  (soft)     ║
║  7. backend-test              (HARD)     ║
║  8. backend-performance       (soft)     ║
║  9. backend-deploy            (HARD)     ║
╚══════════════════════════════════════════╝
```

## Stage invocation

| Stage | Sub-step | Agent file | Phase | Mode |
|-------|---------|-----------|-------|------|
| 1 | Staff | `.cursor/agents/specialists/lld-requirements.md` | Design | read-only |
| 1 | Review | `.cursor/agents/specialists/lld-principal-reviewer.md` | Design | read-only |
| 1 | Respond | `.cursor/agents/specialists/lld-requirements.md` | Design | read-only |
| 2 | Staff | `.cursor/agents/specialists/lld-api-designer.md` | Design | read-only |
| 2 | Review | `.cursor/agents/specialists/lld-principal-reviewer.md` | Design | read-only |
| 2 | Respond | `.cursor/agents/specialists/lld-api-designer.md` | Design | read-only |
| 3 | Staff | `.cursor/agents/specialists/lld-data-modeler.md` | Design | read-only |
| 3 | Review | `.cursor/agents/specialists/lld-principal-reviewer.md` | Design | read-only |
| 3 | Respond | `.cursor/agents/specialists/lld-data-modeler.md` | Design | read-only |
| 4 | Staff | `.cursor/agents/specialists/lld-sequence-flows.md` | Design | read-only |
| 4 | Review | `.cursor/agents/specialists/lld-principal-reviewer.md` | Design | read-only |
| 4 | Respond | `.cursor/agents/specialists/lld-sequence-flows.md` | Design | read-only |
| 5 | Staff | `.cursor/agents/specialists/lld-trade-offs.md` | Design | read-only |
| 5 | Review | `.cursor/agents/specialists/lld-principal-reviewer.md` | Design | read-only |
| 5 | Respond | `.cursor/agents/specialists/lld-trade-offs.md` | Design | read-only |
| 6 | — | `.cursor/agents/specialists/backend-design-validator.md` | Build | read-only |
| 7 | — | `.cursor/agents/specialists/backend-test.md` | Build | shell |
| 8 | — | `.cursor/agents/specialists/backend-performance.md` | Build | shell |
| 9 | — | `.cursor/agents/specialists/backend-deploy.md` | Build | shell |

Pass each stage's **revised** output (post-response) as context to the next stage. For stage 6, pass the complete revised LLD from stages 1c–5c as the design reference.

## Gate rules

| Gate | Type | Condition |
|------|------|-----------|
| Design gate (after stage 5c) | **Hard** | Phase 2 does not start if: design confidence < 70%, OR any Unresolved Blocking challenge exists, OR any Required HITL is pending |
| Test (stage 7) | **Hard** | FAIL or BLOCKED → do not run stages 8 or 9 |
| Deploy (stage 9) | **Hard** | FAILED → report exact rollback steps |
| Design validator (stage 6) | Soft | NEEDS CHANGES → warn user; log Critical items |
| Performance (stage 8) | Soft | Critical bottleneck → warn user before stage 9 |

## HITL rules

- Do not enter Phase 2 with any Unresolved Blocking challenge or pending Required HITL from Phase 1.
- Do not deploy with any pending Required HITL from Phase 2, unless user explicitly overrides.
- **Pipeline confidence** = minimum confidence across all Staff + Review sub-steps in stages 1–9.
- Final verdict **READY TO SHIP** requires pipeline confidence ≥ 70% and zero pending Required HITL.

## Output

```markdown
# Design & Ship — Complete

## Problem
[stated problem]

## Phase 1 — Design confidence dashboard

| Stage | Sub-step | Confidence % | Blocking challenges | Verdict |
|-------|---------|--------------|---------------------|---------|
| 1 Requirements | Staff | | — | — |
| 1 Requirements | Review | | N | APPROVED / NEEDS REVISION |
| 1 Requirements | Respond | | — | — |
| 2 API | Staff | | — | — |
| 2 API | Review | | | |
| 2 API | Respond | | — | — |
| 3 Data model | Staff | | — | — |
| 3 Data model | Review | | | |
| 3 Data model | Respond | | — | — |
| 4 Flows | Staff | | — | — |
| 4 Flows | Review | | | |
| 4 Flows | Respond | | — | — |
| 5 Trade-offs | Staff | | — | — |
| 5 Trade-offs | Review | | | |
| 5 Trade-offs | Respond | | — | — |

**Design confidence**: NN%
**Debate gate**: PASSED / BLOCKED (N unresolved blocking challenges)

## Phase 2 — Build & ship confidence dashboard
| Stage | Agent | Verdict | Confidence % | Required HITL | Pending HITL |
|-------|-------|---------|--------------|---------------|--------------|
| 6 Validate | backend-design-validator | APPROVED / WARN / BLOCK | | | |
| 7 Test | backend-test | PASS / FAIL | | | |
| 8 Performance | backend-performance | OK / WARN | | | |
| 9 Deploy | backend-deploy | SUCCESS / SKIPPED / FAILED | | | |

**Build confidence**: NN%

## Pipeline confidence (overall)
NN% (min across all sub-steps stages 1–9)

## Human review queue (consolidated)
- [ ] [Unresolved Blocking challenge or Required HITL — labeled by stage]

## LLD output
`docs/design/problems/<problem-name>/lld.md`

## Overall verdict
READY TO SHIP | NOT READY | SHIPPED | BLOCKED — design gate | BLOCKED — HITL pending

## Blockers
- [item — stage reference]

## Next steps
1.
```
