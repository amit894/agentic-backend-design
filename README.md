# Backend LLD Kit

Cursor workflow for **developer LLD design rounds** and **backend release validation**. No application code — agents, commands, skills, and design docs only.

## What's included

```
.cursor/
├── agents/specialists/    11 single-job specialist agents
├── workflows/             3 orchestrators (lld-round, backend-release, design-and-ship)
├── skills/                3 routing documents (lld-design-round, backend-release-pipeline, design-and-ship)
├── commands/              4 slash commands
├── PIPELINE-DIAGRAM.md    flowcharts: workflow selection, debate loop, confidence gates, artifact map
├── WORKFLOW-ARCHITECTURE.md  artifact roles, when to create what, new pipeline checklist
└── CONFIDENCE-SCORING.md  HITL rubric used by all agents

docs/design/
├── PROBLEM-BRIEF.md       fill this before running /lld-round
├── LLD.md                 index of active designs
├── LLD-TEMPLATE.md        11-section deliverable structure
├── INTERVIEW-RUBRIC.md    scoring dimensions for self-assessment
└── problems/
    └── <problem-name>/    one folder per design run
        └── lld.md         LLD output written here by the workflow
```

## Quick start

1. Clone or copy this repo
2. Open in Cursor (Agent mode)
3. Fill `docs/design/PROBLEM-BRIEF.md` with your problem
4. Run `/lld-round` in chat
5. Review `docs/design/problems/<problem-name>/lld.md`

## Commands

| Command | What it does | Output |
|---------|-------------|--------|
| `/lld-round` | Full 9-stage LLD design round with Principal Engineer debate | `docs/design/problems/<name>/lld.md` |
| `/design-and-ship` | LLD design round + backend release pipeline end-to-end | LLD doc + release report |
| `/backend-release` | Design validate → test → perf → deploy | Release pipeline report |
| `/extend-workflow` | Add a stage or scaffold a new pipeline | New agents, skill, command |

## How the design round works

Each of the 9 design stages runs a **confidence-gated debate**:

```
Staff Engineer produces output
  │
  ├─ confidence ≥ 90%, no Required/Recommended HITL → AUTO-APPROVED
  │
  └─ otherwise → Principal Engineer challenges (≤ 5 force-ranked)
                       → Staff Engineer responds and revises
```

Stages: Requirements · API · Data Model · Flows · Component Sketch · Trade-offs · Testing Strategy · Observability · Open Questions

See `.cursor/PIPELINE-DIAGRAM.md` for the full visual breakdown.

## Running a single stage

```
Use the lld-api-designer subagent. Problem: [your problem statement].
```

## Mock interview

```
Use the lld-interviewer subagent. I am the candidate. Problem is in docs/design/PROBLEM-BRIEF.md.
```

## Agent inventory

**LLD specialists** (`agents/specialists/`):
`lld-requirements`, `lld-api-designer`, `lld-data-modeler`, `lld-sequence-flows`, `lld-component-sketch`, `lld-trade-offs`, `lld-testing-strategy`, `lld-observability`, `lld-open-questions`, `lld-principal-reviewer`, `lld-interviewer`

**Release specialists** (`agents/specialists/`):
`backend-design-validator`, `backend-test`, `backend-performance`, `backend-deploy`

**Orchestrators** (`workflows/`):
`lld-design-round-workflow`, `backend-release-workflow`, `design-and-ship-workflow`

## Copying into a backend project

```bash
cp -R .cursor docs/design /path/to/your-backend-repo/
```

Agents auto-detect stack from project files. Fill `PROBLEM-BRIEF.md` and run `/lld-round` to produce the design doc before implementing.

## License

MIT
