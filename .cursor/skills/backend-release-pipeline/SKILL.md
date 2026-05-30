---
name: backend-release-pipeline
description: >-
  Runs the full backend release pipeline—design validation, tests, performance
  analysis, and deployment—using project subagents. Use when the user mentions
  backend release workflow, pre-deploy checks, or test validate deploy for
  Java, Python, Node, or Go services. Requires an implementation codebase.
---

# Backend Release Pipeline

Orchestrates four subagents in `.cursor/agents/` against a **backend codebase**.

> This kit is design-first. Run this pipeline after copying agents into an implementation repo, or once application code exists alongside `docs/design/LLD.md`.

## Subagents

| File | Role |
|------|------|
| [backend-design-validator.md](../../agents/backend-design-validator.md) | Architecture, API, security |
| [backend-test.md](../../agents/backend-test.md) | Run and fix tests |
| [backend-performance.md](../../agents/backend-performance.md) | Bottleneck analysis |
| [backend-deploy.md](../../agents/backend-deploy.md) | Build, deploy, verify |
| [backend-release-workflow.md](../../agents/backend-release-workflow.md) | Orchestrator |

## Quick start

```
/backend-release
```

Or:

```
Use the backend-release-workflow subagent. Compare implementation to docs/design/LLD.md if present.
```

## Pipeline order

1. `backend-design-validator` — soft gate (cross-check LLD if available)
2. `backend-test` — **hard gate**
3. `backend-performance` — soft gate
4. `backend-deploy` — hard gate on failure
