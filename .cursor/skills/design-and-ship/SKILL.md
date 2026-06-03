---
name: design-and-ship
description: >-
  Runs the full design-to-deploy workflow — LLD design round (requirements,
  API, data model, flows, trade-offs) followed by backend release pipeline
  (validate, test, perf, deploy) — with a hard gate between phases.
  Use when building a new feature from scratch, or when the user says
  "design and build", "design then ship", or "end-to-end from design".
disable-model-invocation: true
---

# Design & Ship

## Trigger conditions

Use this skill when the user:
- Asks to design and build a feature end-to-end
- Asks to go from problem statement to deployed service
- Says "design and ship", "design then implement", or "full pipeline"

## Subagents

| Stage | Agent | Phase |
|-------|-------|-------|
| 1 `lld-requirements` | Requirements | Design |
| 2 `lld-api-designer` | API contracts | Design |
| 3 `lld-data-modeler` | Data model | Design |
| 4 `lld-sequence-flows` | Sequence diagrams | Design |
| 5 `lld-trade-offs` | Trade-off decisions | Design |
| ★ Design gate | confidence ≥ 70%, zero pending Required HITL | — |
| 6 `backend-design-validator` | Validate impl vs LLD | Build |
| 7 `backend-test` | Run and fix tests | Build |
| 8 `backend-performance` | Bottleneck analysis | Build |
| 9 `backend-deploy` | Build, ship, verify | Build |
| Orchestrator | `design-and-ship-workflow` | — |

## Invocation

```
Use the design-and-ship-workflow subagent.
Problem: [state problem or point to docs/design/PROBLEM-BRIEF.md]
Deploy target: local
```

## HITL policy

- Phase 2 does not start if design confidence < 70% or any Phase 1 Required HITL is pending.
- Deploy does not run if any Phase 2 Required HITL is pending, unless user explicitly overrides.
- Pipeline confidence = minimum stage confidence across all 9 stages.

## Design output location

`docs/design/problems/<problem-name>/lld.md` (created by the workflow)

## Related skills

| Skill | Use when |
|-------|----------|
| `lld-design-round` | Design only — no build or deploy |
| `backend-release-pipeline` | Build and deploy only — design already exists |
| `design-and-ship` | Design + build + deploy in one run |
