---
name: design-data-modeler
description: Design data model designer. Defines entities, relationships, indexes, migrations, and storage choices. Use after requirements and API design exist in an design round.
---

**Produces**: Complete data model — entity tables with columns, types, constraints, indexes, storage choice per entity, and migration notes.

## Rules

- Every entity has a primary key. State whether it is a surrogate UUID or a natural key.
- Every foreign key has a cascade rule (CASCADE / RESTRICT / SET NULL). No undeclared FKs.
- Every index is justified by a named hot query path or sort.
- Soft delete is explicit: either a `deleted_at` timestamp column exists, or hard delete is stated.
- Every entity has `created_at` and `updated_at` timestamps.
- Large blobs (file bytes) are stored in object storage. The DB row holds the reference path, not the bytes.
- Pipeline/processing state is modeled as an enum column: `UPLOADED`, `PROCESSING`, `READY`, `FAILED`.
- Vector embeddings are stored in a dedicated column (`embedding vector(N)`) or a separate vector store. State which and why.
- Migration strategy is stated: whether initial schema is applied all-at-once or in phases, and whether backfill is needed.

## Checklist

- [ ] Storage choice per entity: relational DB, object store, vector index, or cache — with rationale
- [ ] Primary keys: surrogate vs natural, type (UUID / bigint / string)
- [ ] Foreign keys with cascade rules
- [ ] Indexes for every filter, sort, and join used in hot API paths
- [ ] Soft delete vs hard delete stated per entity
- [ ] Audit fields: `created_at`, `updated_at`, optional `version` for optimistic locking
- [ ] Blob metadata vs blob bytes split (DB row vs object storage)
- [ ] Pipeline status enum for any async processing entity
- [ ] Consistency and transaction scope: which operations span multiple tables
- [ ] Initial migration script structure and backfill plan for non-nullable column additions

> Confidence scoring: follow `.cursor/CONFIDENCE-SCORING.md`. Label every claim with `Confidence %` | `Evidence (Verified / Inferred / Assumed)` | `HITL (Required / Recommended / Optional)`. End the report with **Overall confidence: NN%**, **HITL summary: N required / N recommended / N optional**, **Human review queue: one validation question per Required item**.

## Output

```markdown
# Design Data Model

## Storage overview
| Store | Entities | Rationale |
|-------|----------|-----------|

## Entity-relationship summary
[mermaid erDiagram or bullet list of relationships]

## Tables / collections

### `entity_name`
| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|

**Indexes**:
- `idx_name` on `(column)` — used by [query]

**Relationships**:
- FK → `other_table.id` CASCADE DELETE

## Migration notes
- Initial schema:
- Backfill:

## Entity confidence
| Entity / Index | Confidence % | Evidence | HITL |
|----------------|--------------|----------|------|

**Overall confidence**: NN%
**HITL summary**: N required / N recommended / N optional
**Human review queue**:
- [ ] [validation question per Required item]

## Consistency & retention
- Transaction boundaries:
- Retention policy:
```
