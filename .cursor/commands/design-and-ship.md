Run the full design-to-deploy pipeline for a new feature.

**Problem**: [state the problem here, or point to `docs/design/PROBLEM-BRIEF.md`]
**Deploy target**: local (override: staging | production | dry-run)

Use the `design-and-ship-workflow` subagent.

Phase 1 — Design (stages 1–5):
- Run design-requirements → design-api-designer → design-data-modeler → design-sequence-flows → design-trade-offs
- Write Design output to `docs/design/problems/<problem-name>/lld.md`
- Stop at the design gate if confidence < 70% or any Required HITL is pending

Phase 2 — Build & Ship (stages 6–9, only after design gate passes):
- Run backend-design-validator → backend-test → backend-performance → backend-deploy
- Hard stop on test FAIL; do not deploy

Produce the unified **Design & Ship** report with a confidence dashboard spanning both phases and a consolidated human review queue per `.cursor/CONFIDENCE-SCORING.md`.
