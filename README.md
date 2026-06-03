# Backend LLD Kit

Cursor workflow for **developer LLD design rounds** and **backend release validation**. No application code — agents, commands, skills, and design docs only.

## What's included

```
.cursor/
├── agents/          # 12 subagents (LLD pipeline + release pipeline)
├── commands/        # /lld-round, /backend-release, /extend-workflow
├── skills/          # lld-design-round, backend-release-pipeline, agentic-workflows
├── WORKFLOW-ARCHITECTURE.md   artifact roles and new pipeline checklist
└── CONFIDENCE-SCORING.md      HITL rubric used by all agents

docs/design/
├── PROBLEM-BRIEF.md   fill this before running /lld-round
├── LLD.md             LLD workflow output (populated by /lld-round)
├── LLD-TEMPLATE.md    11-section deliverable structure
└── INTERVIEW-RUBRIC.md  scoring dimensions for self-assessment
```

## Quick start

1. Clone or copy this repo
2. Open in Cursor (Agent mode)
3. Fill `docs/design/PROBLEM-BRIEF.md` with your problem
4. Run `/lld-round` in chat
5. Review `docs/design/LLD.md`

## Commands

| Command | What it does | Output |
|---------|-------------|--------|
| `/lld-round` | Full LLD design round | `docs/design/LLD.md` |
| `/backend-release` | Design → test → perf → deploy | Release pipeline report |
| `/extend-workflow` | Add a stage or create a new pipeline | New agents, skill, command |

## Running a single stage

```
Use the lld-api-designer subagent. Problem: [your problem statement].
```

## Mock interview

```
Use the lld-interviewer subagent. I am the candidate. Problem is in docs/design/PROBLEM-BRIEF.md.
```

## Copying into a backend project

```bash
cp -R .cursor docs/design /path/to/your-backend-repo/
```

The agents auto-detect stack from project files. `PROBLEM-BRIEF.md` and `LLD.md` become the design source of truth during implementation.

## Agent inventory

**LLD pipeline**: `lld-requirements`, `lld-api-designer`, `lld-data-modeler`, `lld-sequence-flows`, `lld-trade-offs`, `lld-interviewer`, `lld-design-round-workflow`

**Release pipeline**: `backend-design-validator`, `backend-test`, `backend-performance`, `backend-deploy`, `backend-release-workflow`

## Architecture

- `.cursor/PIPELINE-DIAGRAM.md` — flowcharts: workflow selection, per-stage debate loop, confidence gates, full artifact map
- `.cursor/WORKFLOW-ARCHITECTURE.md` — artifact roles, when to create what, new pipeline checklist

## License

MIT
