---
name: backend-release-pipeline
description: >-
  Runs the full backend release pipeline — design validation, tests, performance
  analysis, and deployment — using project subagents. Use when the user mentions
  backend release, pre-deploy checks, test-validate-deploy, or end-to-end backend
  validation for Java, Python, Node, or Go services.
disable-model-invocation: true
---

# Backend Release Pipeline

## Trigger conditions

Use this skill when the user:
- Asks to release, deploy, or ship the backend
- Asks for pre-merge or pre-deploy validation
- Asks to run tests and deploy
- Asks for an end-to-end backend check

## Subagents

| Agent file | Role | Gate |
|-----------|------|------|
| `backend-design-validator.md` | Architecture, API, security review | Soft |
| `backend-test.md` | Run and fix automated tests | **Hard** |
| `backend-performance.md` | Find bottlenecks with evidence | Soft |
| `backend-deploy.md` | Build, deploy, verify, rollback | Hard on failure |
| `backend-release-workflow.md` | Full pipeline orchestrator | — |

## Invocation

**Full pipeline:**
```
Use the backend-release-workflow subagent. Deploy target: local.
```

**Single stage:**
```
Use the backend-test subagent. Fix all failures.
```

**Specific scope:**
```
Use the backend-release-workflow subagent. Scope: [module name]. Target: dry-run.
```

## Stage selection by intent

| User intent | Stages |
|-------------|--------|
| Pre-merge review | 1 + 2 |
| Pre-production checklist | 1 + 2 + 3 |
| Ship locally | 1 + 2 + 3 + 4 (target: local) |
| CI-only validation | 1 + 2 + 3 (skip deploy) |

## HITL policy

- Do not deploy with any pending Required HITL item unless the user explicitly overrides.
- Pipeline confidence = minimum stage confidence across completed stages.
- READY TO SHIP requires pipeline confidence ≥ 70% and zero pending Required HITL.
