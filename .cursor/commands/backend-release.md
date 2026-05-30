Run the full backend release pipeline for this repository.

**Prerequisite**: A backend codebase must exist in this workspace (this kit is often copied into an implementation repo). If no code is present, run design-only and skip deploy.

Stages (in order):
1. **backend-design-validator** — review architecture, APIs, security (compare to `docs/design/LLD.md` if present)
2. **backend-test** — run the test suite; fix failures (hard gate)
3. **backend-performance** — identify bottlenecks with evidence
4. **backend-deploy** — build and deploy locally (Docker Compose or project default)

Use the `backend-release-workflow` subagent to orchestrate. Default deploy target: **local**. Stop before deploy if tests fail.

Produce the consolidated **Backend Release Pipeline Report** when complete.
