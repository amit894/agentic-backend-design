---
name: lld-design-round
description: >-
  Runs a full developer LLD (low-level design) round—requirements, API, data
  model, flows, trade-offs, mock interview, and code alignment. Use for backend
  or LLM feature interview prep, greenfield design docs, or comparing design to
  implementation. Reads problem statement from docs/design/PROBLEM-BRIEF.md.
---

# LLD Design Round

Standalone design kit — no application code required. Fill [PROBLEM-BRIEF.md](../../../docs/design/PROBLEM-BRIEF.md), then run the pipeline.

## Subagents (`.cursor/agents/`)

| Agent | Stage |
|-------|-------|
| `lld-requirements` | FR/NFR, scope |
| `lld-api-designer` | API contracts |
| `lld-data-modeler` | Schema & storage |
| `lld-sequence-flows` | Sequence diagrams |
| `lld-trade-offs` | ADR-style decisions |
| `lld-interviewer` | Mock interview |
| `lld-design-round-workflow` | Orchestrator |

## Quick start

1. Edit `docs/design/PROBLEM-BRIEF.md`
2. In Agent chat: `/lld-round`
3. Output lands in `docs/design/LLD.md`

## Bridge to implementation

After implementing in a separate repo (or after adding code here):

```
Use backend-release-workflow to test, validate, profile, and deploy.
```

| Phase | Command |
|-------|---------|
| Design | `/lld-round` |
| Build & ship | `/backend-release` |

## Artifacts (`docs/design/`)

| File | Purpose |
|------|---------|
| [PROBLEM-BRIEF.md](../../../docs/design/PROBLEM-BRIEF.md) | **Start here** — problem statement input |
| [LLD.md](../../../docs/design/LLD.md) | Living LLD output |
| [LLD-TEMPLATE.md](../../../docs/design/LLD-TEMPLATE.md) | Section structure reference |
| [INTERVIEW-RUBRIC.md](../../../docs/design/INTERVIEW-RUBRIC.md) | Scoring dimensions |

## Use in another repo

Copy the entire repo or merge `.cursor/` + `docs/design/` into your backend project. Keep `PROBLEM-BRIEF.md` and `LLD.md` in sync with implementation.
