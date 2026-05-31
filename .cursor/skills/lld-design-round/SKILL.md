---
name: lld-design-round
description: >-
  Runs a full developer LLD (low-level design) round—requirements, API, data
  model, flows, trade-offs, mock interview, and code alignment. Use for backend
  or LLM feature interview prep, greenfield design docs, or comparing design to
  implementation.
---

# LLD Design Round

## Human-in-the-loop confidence

Each stage reports **Confidence %**, **Evidence**, and **HITL** per [.cursor/CONFIDENCE-SCORING.md](../../CONFIDENCE-SCORING.md). The orchestrator produces a **Confidence dashboard** and **Human review queue**. Resolve all **Required** HITL before approving design or running `/backend-release`.

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

## Bridge to implementation pipeline

After LLD is approved:

```
Use backend-release-workflow to test, validate, profile, and deploy.
```

| Phase | Skill / command |
|-------|-----------------|
| Design | `/lld-round` or `lld-design-round-workflow` |
| Build & ship | `/backend-release` or `backend-release-workflow` |

## Artifacts

| File | Purpose |
|------|---------|
| [CONFIDENCE-SCORING.md](../../CONFIDENCE-SCORING.md) | HITL confidence rubric |

## Design docs (`docs/design/`)

| File | Purpose |
|------|---------|
| [PROBLEM-BRIEF.md](../../../docs/design/PROBLEM-BRIEF.md) | **Start here** — problem input |
| [LLD.md](../../../docs/design/LLD.md) | Living LLD output |
| [LLD-TEMPLATE.md](../../../docs/design/LLD-TEMPLATE.md) | Section structure |
| [INTERVIEW-RUBRIC.md](../../../docs/design/INTERVIEW-RUBRIC.md) | Scoring dimensions |

## Optional additions (not scaffolded)

- `docs/design/ADR-TEMPLATE.md` — one decision per file
- `docs/design/HLD.md` — system context (only if round includes HLD)
- `.cursor/rules/lld-docs.mdc` — auto-apply when editing `docs/design/**`
