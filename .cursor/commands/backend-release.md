Run the full backend release pipeline for this repository.

Use the `backend-release-workflow` subagent.

Stages (in order):
1. `backend-design-validator` — architecture, API, security review (soft gate)
2. `backend-test` — run tests and fix failures (**hard gate**: do not proceed to stage 3 on FAIL)
3. `backend-performance` — identify bottlenecks with evidence (soft gate)
4. `backend-deploy` — build and deploy to **local** via Docker Compose or project default

Default deploy target: **local**. Override: state `staging` or `production` or `dry-run` in the prompt.

Produce the consolidated **Backend Release Pipeline Report** including a confidence dashboard and human review queue. Block deploy on any pending Required HITL per `.cursor/CONFIDENCE-SCORING.md`.
