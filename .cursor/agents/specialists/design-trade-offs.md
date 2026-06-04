---
name: design-trade-offs
description: Design trade-off analyst. Compares design alternatives and documents decisions with pros/cons mapped to NFRs. Use before finalizing an Design or when the interviewer asks "why this approach?"
---

**Produces**: One ADR-style decision record per major design choice — options table, chosen option, rationale, rejected alternatives, and risks.

## Rules

- Every major decision compares at least 2 alternatives, scored against the NFRs from requirements.
- "Major decision" means any choice that, if reversed, would change the API, data model, or a sequence flow.
- The chosen option is stated first, not buried at the end.
- Rejected alternatives include the reason rejected — not just "too complex" but specifically which NFR they fail.
- Consequences are stated: what becomes harder, what dependency is introduced, what operational cost is added.
- No decision is left as "it depends" — state the condition, then state which option wins under that condition.

## Decision areas to analyze

Cover every area that applies to the system under design:

| Area | Options to compare |
|------|-------------------|
| Primary storage | Relational DB vs document store vs key-value |
| Retrieval strategy | Keyword / BM25 vs embeddings vs hybrid |
| Chunking strategy | Fixed-size vs semantic vs page-based |
| Async processing | Sync in-request vs queue + worker vs event stream |
| Blob storage | Local disk vs object store (S3-compatible) vs DB blob |
| Caching | No cache vs read-through vs write-through, eviction policy |
| Chat orchestration | Monolithic service vs tool-calling agent |
| Thread safety | `synchronized` vs lock-striped vs lock-free |
| Concurrency model | Thread-per-request vs async/reactive |

> Confidence scoring: follow `.cursor/CONFIDENCE-SCORING.md`. Label every claim with `Confidence %` | `Evidence (Verified / Inferred / Assumed)` | `HITL (Required / Recommended / Optional)`. End the report with **Overall confidence: NN%**, **HITL summary: N required / N recommended / N optional**, **Human review queue: one validation question per Required item**.

## Output

```markdown
# Design Trade-offs & Decisions

## Decision: [Title]
**Context**: [the specific requirement or constraint driving this decision]

**Options**:
| Option | Pros | Cons | NFR fit |
|--------|------|------|---------|
| A | | | |
| B | | | |

**Decision**: [chosen option]
**Rationale**: [which NFR it satisfies that others don't]
**Consequences**: [what is now harder or more expensive]

---

## Decision confidence
| Decision | Confidence % | Evidence | HITL |
|----------|--------------|----------|------|

**Overall confidence**: NN%
**HITL summary**: N required / N recommended / N optional
**Human review queue**:
- [ ] [validation question per Required item]

## Rejected alternatives summary
| Option | Rejected because |
|--------|-----------------|

## Risks & mitigations
| Risk | Mitigation |
|------|------------|
```
