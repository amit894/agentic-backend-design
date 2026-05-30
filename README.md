# Backend LLD Kit

Standalone Cursor workflow for **developer low-level design (LLD) rounds** and **backend release validation**. No application code — only agents, commands, skills, and design docs.

Use this repo to start a new problem statement, practice interviews, or copy `.cursor/` + `docs/design/` into an implementation project.

## What's included

```
.cursor/
├── agents/          # 12 subagents (LLD + release pipeline)
├── commands/        # /lld-round, /backend-release
└── skills/          # Orchestration guides

docs/design/
├── PROBLEM-BRIEF.md # ← fill this first
├── LLD.md           # ← workflow output
├── LLD-TEMPLATE.md  # section reference
└── INTERVIEW-RUBRIC.md
```

## Quick start

1. **Clone or copy** this repo
2. **Open in Cursor** (Agent mode)
3. **Edit** `docs/design/PROBLEM-BRIEF.md` with your problem
4. **Run** `/lld-round` in chat
5. **Review** `docs/design/LLD.md`

### Mock interview

```
Use the lld-interviewer subagent — I'm the candidate. Problem is in PROBLEM-BRIEF.md.
```

### After you implement code

Copy this kit into your backend repo (or add code here), then:

```
/backend-release
```

## Workflows

| Workflow | Command | Output |
|----------|---------|--------|
| LLD design round | `/lld-round` | `docs/design/LLD.md` |
| Release pipeline | `/backend-release` | Test + perf + deploy report |
| Single stage | `Use the lld-api-designer subagent` | Stage-specific report |

## Copy into a backend project

```bash
cp -R .cursor docs/design /path/to/your-backend-repo/
```

Keep `PROBLEM-BRIEF.md` and `LLD.md` as the design source of truth while implementing.

## Agents

**LLD:** `lld-requirements`, `lld-api-designer`, `lld-data-modeler`, `lld-sequence-flows`, `lld-trade-offs`, `lld-interviewer`, `lld-design-round-workflow`

**Release:** `backend-design-validator`, `backend-test`, `backend-performance`, `backend-deploy`, `backend-release-workflow`

## License

MIT — use freely for interview prep and project scaffolding.
