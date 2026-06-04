---
name: design-round
description: >-
  Runs a full developer design round — requirements, API, data model, flows,
  trade-offs, optional mock interview, optional code gap analysis. Use for backend
  or LLM feature interview prep, greenfield design docs, or comparing a design to
  an existing implementation.
disable-model-invocation: true
---

# Design Design Round

## Trigger conditions

Use this skill when the user:
- Asks to run an design round, design interview, or design session for any backend system
- Asks to produce a design doc, API spec, data model, or sequence diagram for a new feature
- Asks to compare a written design to an existing codebase

## Subagents

| Agent | Stage | Output |
|-------|-------|--------|
| `design-requirements` | 1a Staff | FR/NFR table, assumptions, out-of-scope |
| `design-principal-reviewer` | 1b Review | Force-ranked challenges, severity, verdict |
| `design-requirements` | 1c Respond | Revised FR/NFR addressing challenges |
| `design-api-designer` | 2a Staff | Endpoint contracts, error model, auth |
| `design-principal-reviewer` | 2b Review | API challenges |
| `design-api-designer` | 2c Respond | Revised API contracts |
| `design-data-modeler` | 3a Staff | Entity tables, indexes, storage choices |
| `design-principal-reviewer` | 3b Review | Data model challenges |
| `design-data-modeler` | 3c Respond | Revised data model |
| `design-sequence-flows` | 4a Staff | Mermaid sequence diagrams, component map |
| `design-principal-reviewer` | 4b Review | Flow challenges |
| `design-sequence-flows` | 4c Respond | Revised flows |
| `design-component-sketch` | 5a Staff | Component responsibility table, interfaces, class sketch |
| `design-principal-reviewer` | 5b Review | Component sketch challenges |
| `design-component-sketch` | 5c Respond | Revised component sketch |
| `design-trade-offs` | 6a Staff | ADR-style decisions with rejected alternatives |
| `design-principal-reviewer` | 6b Review | Trade-off challenges |
| `design-trade-offs` | 6c Respond | Revised decisions |
| `design-testing-strategy` | 7a Staff | Test matrix, named test cases, coverage targets |
| `design-principal-reviewer` | 7b Review | Testing strategy challenges |
| `design-testing-strategy` | 7c Respond | Revised testing strategy |
| `design-observability` | 8a Staff | Metrics, logs, alerts, health checks, SLO |
| `design-principal-reviewer` | 8b Review | Observability challenges |
| `design-observability` | 8c Respond | Revised observability plan |
| `design-open-questions` | 9a Staff | Resolution table for all PROBLEM-BRIEF questions |
| `design-principal-reviewer` | 9b Review | Open questions resolution challenges |
| `design-open-questions` | 9c Respond | Final resolution table |
| `design-interviewer` | 10 (optional) | Mock interview score and hire recommendation |
| `design-round-workflow` | Orchestrator | Merged Design doc, debate log, confidence dashboard |

## Invocation

```
Use the design-round-workflow subagent.
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
| `docs/design/problems/<name>/lld.md` | Design output per problem |
| `docs/design/Design-TEMPLATE.md` | 11-section deliverable structure |
| `docs/design/INTERVIEW-RUBRIC.md` | Scoring dimensions for self-assessment |
| `.cursor/CONFIDENCE-SCORING.md` | HITL confidence rubric |

## Bridge to implementation

After Design is approved → run `/backend-release` or invoke `backend-release-workflow`.
