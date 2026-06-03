---
name: lld-design-round
description: >-
  Runs a full developer LLD design round — requirements, API, data model, flows,
  trade-offs, optional mock interview, optional code gap analysis. Use for backend
  or LLM feature interview prep, greenfield design docs, or comparing a design to
  an existing implementation.
disable-model-invocation: true
---

# LLD Design Round

## Trigger conditions

Use this skill when the user:
- Asks to run an LLD round, design interview, or design session for any backend system
- Asks to produce a design doc, API spec, data model, or sequence diagram for a new feature
- Asks to compare a written design to an existing codebase

## Subagents

| Agent | Stage | Output |
|-------|-------|--------|
| `lld-requirements` | 1 | FR/NFR table, assumptions, out-of-scope |
| `lld-api-designer` | 2 | Endpoint contracts, error model, auth |
| `lld-data-modeler` | 3 | Entity tables, indexes, storage choices |
| `lld-sequence-flows` | 4 | Mermaid sequence diagrams, component map |
| `lld-trade-offs` | 5 | ADR-style decisions with rejected alternatives |
| `lld-interviewer` | 6 (optional) | Mock interview score and hire recommendation |
| `lld-design-round-workflow` | Orchestrator | Merged LLD doc, confidence dashboard |

## Invocation

```
Use the lld-design-round-workflow subagent.
Problem: [state problem or point to docs/design/PROBLEM-BRIEF.md]
Output to: docs/design/problems/<problem-name>/lld.md
```

## HITL policy

- Do not mark design Approved while any stage 1–5 has a pending Required HITL item.
- Pipeline confidence = minimum stage confidence across stages 1–5.
- Resolve all Required items before running `/backend-release`.

## Design documents

| File | Purpose |
|------|---------|
| `docs/design/PROBLEM-BRIEF.md` | Fill before running — problem input |
| `docs/design/problems/<name>/lld.md` | LLD output per problem |
| `docs/design/LLD-TEMPLATE.md` | 11-section deliverable structure |
| `docs/design/INTERVIEW-RUBRIC.md` | Scoring dimensions for self-assessment |
| `.cursor/CONFIDENCE-SCORING.md` | HITL confidence rubric |

## Bridge to implementation

After LLD is approved → run `/backend-release` or invoke `backend-release-workflow`.
