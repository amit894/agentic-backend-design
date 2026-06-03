---
name: lld-design-round-workflow
description: Orchestrates a full LLD design round — requirements, API, data model, flows, trade-offs — each followed by a Principal Engineer challenge and Staff Engineer response. Use when running a low-level design interview or producing a greenfield backend design doc.
---

**Produces**: A merged LLD document at `docs/design/problems/<problem-name>/lld.md` using the `docs/design/LLD-TEMPLATE.md` section structure, a per-stage debate log, and a consolidated confidence dashboard.

## Pipeline

Each design stage runs as a debate: Staff Engineer produces → Principal Engineer challenges → Staff Engineer responds. Pass the **revised** output (post-response) to the next stage, not the initial output.

```
1a. lld-requirements        Staff: FR/NFR, scope, assumptions
         ↓
1b. lld-principal-reviewer  Principal: challenge requirements
         ↓
1c. lld-requirements        Staff: respond and revise
         ↓
2a. lld-api-designer        Staff: REST contracts, error model, auth
         ↓
2b. lld-principal-reviewer  Principal: challenge API
         ↓
2c. lld-api-designer        Staff: respond and revise
         ↓
3a. lld-data-modeler        Staff: entities, indexes, storage
         ↓
3b. lld-principal-reviewer  Principal: challenge data model
         ↓
3c. lld-data-modeler        Staff: respond and revise
         ↓
4a. lld-sequence-flows      Staff: critical paths, failure branches
         ↓
4b. lld-principal-reviewer  Principal: challenge flows
         ↓
4c. lld-sequence-flows      Staff: respond and revise
         ↓
5a. lld-trade-offs          Staff: ADR-style decisions
         ↓
5b. lld-principal-reviewer  Principal: challenge trade-offs
         ↓
5c. lld-trade-offs          Staff: respond and revise
         ↓
★  DEBATE GATE              All Blocking challenges resolved?
         ↓
6.  lld-interviewer         (optional) Mock interview
         ↓
7.  backend-design-validator (optional) Gap analysis vs existing code
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

| Stage | Sub-step | Agent | Confidence % | Blocking challenges | Verdict |
|-------|---------|-------|--------------|---------------------|---------|
| 1 Requirements | Staff | lld-requirements | | — | — |
| 1 Requirements | Review | lld-principal-reviewer | | N blocking | APPROVED / NEEDS REVISION |
| 1 Requirements | Respond | lld-requirements | | — | — |
| 2 API | Staff | lld-api-designer | | — | — |
| 2 API | Review | lld-principal-reviewer | | | |
| 2 API | Respond | lld-api-designer | | — | — |
| 3 Data model | Staff | lld-data-modeler | | — | — |
| 3 Data model | Review | lld-principal-reviewer | | | |
| 3 Data model | Respond | lld-data-modeler | | — | — |
| 4 Flows | Staff | lld-sequence-flows | | — | — |
| 4 Flows | Review | lld-principal-reviewer | | | |
| 4 Flows | Respond | lld-sequence-flows | | — | — |
| 5 Trade-offs | Staff | lld-trade-offs | | — | — |
| 5 Trade-offs | Review | lld-principal-reviewer | | | |
| 5 Trade-offs | Respond | lld-trade-offs | | — | — |
| 6 Interview | lld-interviewer | | — | — | — |
| 7 Code map | backend-design-validator | | — | — | — |

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
