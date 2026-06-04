---
name: design-sequence-flows
description: Design flow designer. Produces Mermaid sequence diagrams and component sketches for critical paths. Use after API and data model exist in a design round.
---

**Produces**: 2–4 Mermaid sequence diagrams covering critical paths, failure branches, and a component responsibility table.

## Rules

- Identify critical flows from the requirements: at minimum one write flow, one read flow, and one failure/recovery flow.
- Diagrams show components (Controller, Service, Repository, external system), not individual methods or variables.
- Every diagram labels the data payload on each arrow, not just the method name.
- Sync vs async boundaries are marked explicitly with a note on the arrow.
- Transaction boundaries are annotated with `Note over Service: begin tx` / `end tx`.
- Timeout and retry behavior is shown on external call arrows, not described in prose.
- Failure branches are separate diagrams or `alt/else` blocks — not omitted.
- Component names match the layer naming used in the repo or agreed design: Controller / Handler, Service / UseCase, Repository / Store, and named external systems.

## Flows to cover

Cover all flows that apply to the system being designed:

| Flow | Required when |
|------|---------------|
| Write / ingest happy path | Any mutation operation |
| Read / query happy path | Any retrieval operation |
| Async processing (background job / queue) | Any operation with a worker or queue |
| Failure and retry | Every external call or async boundary |
| Auth / token validation | Any authenticated endpoint |

## Checklist

- [ ] Component map: list every module with its single responsibility
- [ ] Happy-path write flow with transaction boundary annotated
- [ ] Happy-path read flow with cache hit and cache miss branches
- [ ] Failure flow: external call timeout or error → retry / fallback / client error
- [ ] Async boundary: producer → queue → consumer handoff
- [ ] Idempotency enforcement shown at the service layer
- [ ] Data payload labeled on every arrow

> Confidence scoring: follow `.cursor/CONFIDENCE-SCORING.md`. Label every claim with `Confidence %` | `Evidence (Verified / Inferred / Assumed)` | `HITL (Required / Recommended / Optional)`. End the report with **Overall confidence: NN%**, **HITL summary: N required / N recommended / N optional**, **Human review queue: one validation question per Required item**.

## Output

```markdown
# Design Sequence & Component Flows

## Component map
| Component | Responsibility |
|-----------|----------------|

## Flow 1: [Name]
```mermaid
sequenceDiagram
  participant Client
  participant Controller
  participant Service
  participant Repository
  participant DB
  Client->>Controller: POST /resource {payload}
  ...
```
**Transaction boundary**: [describe]
**Timeout / retry**: [describe]

## Flow 2: [Name]
...

## Flow N: Failure — [scenario]
...

## Flow confidence
| Flow | Confidence % | Evidence | HITL |
|------|--------------|----------|------|

**Overall confidence**: NN%
**HITL summary**: N required / N recommended / N optional
**Human review queue**:
- [ ] [validation question per Required item]

## Failure & edge cases
| Flow | Failure scenario | Behavior |
|------|-----------------|----------|
```
