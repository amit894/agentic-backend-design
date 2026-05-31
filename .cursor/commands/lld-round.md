Run a full developer LLD design round for this project.

**Problem**: Document upload + grounded chat backend (or use the user's stated problem).

Stages:
1. `lld-requirements` — FR/NFR, scope, assumptions
2. `lld-api-designer` — REST contracts and error model
3. `lld-data-modeler` — entities, indexes, storage
4. `lld-sequence-flows` — upload, chat, failure paths
5. `lld-trade-offs` — retrieval, chunking, LLM integration decisions
6. *(Optional)* `lld-interviewer` — 3 probing questions on weak areas

Use `lld-design-round-workflow` to orchestrate. Merge output into `docs/design/LLD-TEMPLATE.md` structure.

If this repo already has code, finish with `backend-design-validator` to list design-vs-implementation gaps.

Do not write production code unless asked — design deliverable only.

Include per-stage and pipeline **confidence scores** and a **Human review queue** (Required HITL) per `.cursor/CONFIDENCE-SCORING.md`.
