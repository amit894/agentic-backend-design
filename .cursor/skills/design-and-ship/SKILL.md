---
name: design-and-ship
description: >-
  Runs the full design-to-deploy workflow — LLD design round with per-stage
  Principal Engineer debate (requirements, API, data model, flows, trade-offs)
  followed by backend release pipeline (validate, test, perf, deploy) — with
  a hard gate between phases. Use when building a new feature from scratch,
  or when the user says "design and build", "design then ship", or
  "end-to-end from design".
disable-model-invocation: true
---

# Design & Ship

## Trigger conditions

Use this skill when the user:
- Asks to design and build a feature end-to-end
- Asks to go from problem statement to deployed service
- Says "design and ship", "design then implement", or "full pipeline"

## Debate trigger rule

The Principal Engineer review (Nb) is confidence-gated — it only runs when
Staff output confidence < 90% OR Required/Recommended HITL > 0.
If confidence ≥ 90% with Optional HITL only, the stage is AUTO-APPROVED.

## Subagents

### Phase 1 — Design (confidence-gated debate per stage)

| Stage | Sub-step | Agent | Triggered when |
|-------|---------|-------|---------------|
| 1 Requirements | Staff (Na) | `lld-requirements` | always |
| 1 Requirements | Review (Nb) | `lld-principal-reviewer` | conf < 90% or HITL > 0 |
| 1 Requirements | Respond (Nc) | `lld-requirements` | Nb ran and found challenges |
| 2 API | Staff (Na) | `lld-api-designer` | always |
| 2 API | Review (Nb) | `lld-principal-reviewer` | conf < 90% or HITL > 0 |
| 2 API | Respond (Nc) | `lld-api-designer` | Nb ran and found challenges |
| 3 Data model | Staff (Na) | `lld-data-modeler` | always |
| 3 Data model | Review (Nb) | `lld-principal-reviewer` | conf < 90% or HITL > 0 |
| 3 Data model | Respond (Nc) | `lld-data-modeler` | Nb ran and found challenges |
| 4 Flows | Staff (Na) | `lld-sequence-flows` | always |
| 4 Flows | Review (Nb) | `lld-principal-reviewer` | conf < 90% or HITL > 0 |
| 4 Flows | Respond (Nc) | `lld-sequence-flows` | Nb ran and found challenges |
| 5 Trade-offs | Staff (Na) | `lld-trade-offs` | always |
| 5 Trade-offs | Review (Nb) | `lld-principal-reviewer` | conf < 90% or HITL > 0 |
| 5 Trade-offs | Respond (Nc) | `lld-trade-offs` | Nb ran and found challenges |
| ★ Design gate | — | — | conf ≥ 70%, zero Blocking challenges, zero Required HITL |

### Phase 2 — Build & Ship

| Stage | Agent | Gate |
|-------|-------|------|
| 6 Validate | `backend-design-validator` | Soft |
| 7 Test | `backend-test` | **Hard** |
| 8 Performance | `backend-performance` | Soft |
| 9 Deploy | `backend-deploy` | Hard on failure |
| Orchestrator | `design-and-ship-workflow` | — |

## Invocation

```
Use the design-and-ship-workflow subagent.
Problem: [state problem or point to docs/design/PROBLEM-BRIEF.md]
Deploy target: local
```

## HITL policy

- Phase 2 does not start if design confidence < 70%, any Unresolved Blocking challenge exists, or any Required HITL is pending.
- Deploy does not run if any Phase 2 Required HITL is pending, unless user explicitly overrides.
- Pipeline confidence = minimum stage confidence across all completed sub-steps.

## Design output location

`docs/design/problems/<problem-name>/lld.md` (created by the workflow)

## Related skills

| Skill | Use when |
|-------|----------|
| `lld-design-round` | Design only — no build or deploy |
| `backend-release-pipeline` | Build and deploy only — design already exists |
| `design-and-ship` | Design + build + deploy in one run |
