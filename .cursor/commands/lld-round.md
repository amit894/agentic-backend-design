Run a full developer LLD design round.

**Problem source**: Read `docs/design/PROBLEM-BRIEF.md` first. If empty, ask the user to fill it or state the problem in chat.

Stages:
1. `lld-requirements` — FR/NFR, scope, assumptions
2. `lld-api-designer` — REST contracts and error model
3. `lld-data-modeler` — entities, indexes, storage
4. `lld-sequence-flows` — critical paths and failure handling
5. `lld-trade-offs` — alternatives and ADR-style decisions
6. *(Optional)* `lld-interviewer` — mock interview on weak areas

Use `lld-design-round-workflow` to orchestrate. Write the merged deliverable to `docs/design/LLD.md` using sections from `docs/design/LLD-TEMPLATE.md`.

If a backend codebase exists in this workspace, optionally finish with `backend-design-validator` for design-vs-code gaps.

Design deliverable only — do not write production code unless asked.
