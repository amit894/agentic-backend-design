Run the full backend release pipeline for this repository.

Stages (in order):
1. **backend-design-validator** — review architecture, APIs, security
2. **backend-test** — run the test suite; fix failures (hard gate)
3. **backend-performance** — identify bottlenecks with evidence
4. **backend-deploy** — build and deploy locally via Docker Compose (or project default)

Use the `backend-release-workflow` subagent to orchestrate. Default deploy target: **local**. Stop before deploy if tests fail.

Produce the consolidated **Backend Release Pipeline Report** when complete, including a **Confidence dashboard** and **Human review queue**. Block deploy on pending Required HITL per `.cursor/CONFIDENCE-SCORING.md`.
