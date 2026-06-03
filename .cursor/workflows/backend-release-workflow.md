---
name: backend-release-workflow
description: Orchestrates the full backend release pipeline — design validation, tests, performance analysis, and deployment — in order. Use when the user wants end-to-end backend validation, pre-release checks, or test-and-deploy for any backend project.
---

**Produces**: A consolidated release report with per-stage verdict, confidence dashboard, human review queue, and overall READY TO SHIP / NOT READY / BLOCKED verdict.

## Pipeline

Run stages in order. Do not parallelize stages 1–4. Stop on hard failures.

```
1. backend-design-validator   architecture, API, security review     (soft gate)
        ↓
2. backend-test               unit + integration + API tests          (HARD gate)
        ↓ stop if FAIL
3. backend-performance        bottlenecks, hot paths, load smoke      (soft gate)
        ↓
4. backend-deploy             build, ship, verify, rollback           (HARD gate on failure)
```

## Stage invocation

| Stage | Agent | Notes |
|-------|-------|-------|
| 1 | `.cursor/agents/specialists/backend-design-validator.md` | Full repo context; read-only |
| 2 | `.cursor/agents/specialists/backend-test.md` | Run tests; fix failures; re-run until green or blocked |
| 3 | `.cursor/agents/specialists/backend-performance.md` | Focus on paths changed since last deploy |
| 4 | `.cursor/agents/specialists/backend-deploy.md` | Only after stage 2 PASS; respect user-specified deploy target |

Pass these values to each stage prompt:
- Repo path (workspace root)
- Scope: full backend or specific module
- Prior stage summary: verdict, warnings, changed files
- Deploy target: `local` (default), `staging`, `production`, or `dry-run`

## Gate rules

| Stage | Gate type | Stop condition |
|-------|-----------|---------------|
| Design | Soft | NEEDS CHANGES → ask user before continuing; log all Critical items |
| Test | **Hard** | FAIL or BLOCKED → do not run stages 3 or 4 |
| Performance | Soft | Critical bottleneck → warn user before stage 4 |
| Deploy | **Hard** | FAILED → report exact rollback steps |

## HITL rules

- Collect **Overall confidence** and **Human review queue** from each stage.
- Do not deploy if any stage has a pending **Required** HITL item, unless the user explicitly overrides.
- **Pipeline confidence** = minimum stage confidence across completed stages.
- Mark overall verdict **READY TO SHIP** only when pipeline confidence ≥ 70% and zero pending Required HITL.

## Output

```markdown
# Backend Release Pipeline Report

## Scope
[repo, branch, modules, deploy target]

## Confidence dashboard
| Stage | Agent | Verdict | Confidence % | Required HITL | Pending HITL |
|-------|-------|---------|--------------|---------------|--------------|
| Design | backend-design-validator | APPROVED / WARN / BLOCK | | | |
| Test | backend-test | PASS / FAIL | | | |
| Performance | backend-performance | OK / WARN | | | |
| Deploy | backend-deploy | SUCCESS / SKIPPED / FAILED | | | |

**Pipeline confidence**: NN%

## Stage results
| Stage | Summary |
|-------|---------|
| Design | |
| Test | |
| Performance | |
| Deploy | |

## Human review queue (consolidated)
- [ ] [Required item from each stage — one validation question each]

## Overall verdict
READY TO SHIP | NOT READY | SHIPPED (local / staging / prod) | BLOCKED — HITL pending

## Blockers
- [must-fix items with stage reference]

## Warnings
- [ship-with-caution items]

## Next steps
1.
```
