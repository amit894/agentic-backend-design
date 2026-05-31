---
name: backend-release-pipeline
description: >-
  Runs the full backend release pipeline—design validation, tests, performance
  analysis, and deployment—using project subagents. Use when the user_query
  mentions backend release workflow, pre-deploy checks, test validate deploy,
  or end-to-end backend validation for generic Java, Python, Node, or Go services.
---

# Backend Release Pipeline

Orchestrates four project subagents in `.cursor/agents/` for any backend repository.

## Human-in-the-loop confidence

Each stage reports **Confidence %**, **Evidence**, and **HITL** per [.cursor/CONFIDENCE-SCORING.md](../../CONFIDENCE-SCORING.md). **Do not deploy** with pending **Required** HITL unless the user overrides. See **Confidence dashboard** in `backend-release-workflow` output.

## Subagents

| File | Role |
|------|------|
| [backend-design-validator.md](../../agents/backend-design-validator.md) | Architecture, API, security review |
| [backend-test.md](../../agents/backend-test.md) | Run and fix automated tests |
| [backend-performance.md](../../agents/backend-performance.md) | Find bottlenecks with evidence |
| [backend-deploy.md](../../agents/backend-deploy.md) | Build, deploy, verify, rollback |
| [backend-release-workflow.md](../../agents/backend-release-workflow.md) | Full pipeline orchestrator |

## Quick start

**Full pipeline (recommended):**

```
Use the backend-release-workflow subagent to run the full release pipeline with local dry-run deploy.
```

**Single stage:**

```
Use the backend-test subagent to run all tests and fix failures.
```

## Pipeline order

1. `backend-design-validator` — soft gate
2. `backend-test` — **hard gate** (must pass before deploy)
3. `backend-performance` — soft gate (warn on critical issues)
4. `backend-deploy` — hard gate on deploy failure

## Invocation via Task tool

When orchestrating programmatically, launch one Task per stage with `readonly: true` for design review only.

```text
Stage 1 prompt: "Follow backend-design-validator agent instructions. Repo: {cwd}. Scope: full backend."
Stage 2 prompt: "Follow backend-test agent instructions. Repo: {cwd}. Prior design summary: {stage1}."
Stage 3 prompt: "Follow backend-performance agent instructions. Focus on recent changes. Prior: {stage2}."
Stage 4 prompt: "Follow backend-deploy agent instructions. Target: local. Tests: PASS from stage 2."
```

## Options

| User intent | Stages to run |
|-------------|---------------|
| Pre-merge review | 1 + 2 |
| Pre-prod checklist | 1 + 2 + 3 |
| Ship locally | 1 + 2 + 3 + 4 (target: local) |
| CI-only validation | 1 + 2 + 3 (skip deploy) |

## Copy to other projects

Copy `.cursor/agents/backend-*.md` and `.cursor/skills/backend-release-pipeline/` into any repo. Agents auto-detect stack from project files.
