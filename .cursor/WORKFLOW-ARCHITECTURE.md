# Agentic Workflow Architecture

## Layer model

```
User
  │
  ├─► .cursor/commands/*.md          Slash shortcuts (/lld-round, /backend-release)
  │
  ├─► .cursor/skills/*/SKILL.md      Routing: trigger conditions, agent table, invocation syntax
  │
  └─► .cursor/agents/specialists/*.md and .cursor/agents/workflows/*.md            Specialists and orchestrators — one job, one output format
        │
        └─► Task tool / subagent invocation
```

## Artifact roles

| Artifact | Path | Responsibility |
|----------|------|---------------|
| **Command** | `.cursor/commands/<name>.md` | Problem/intent placeholder + orchestrator name + constraints |
| **Skill** | `.cursor/skills/<name>/SKILL.md` | Trigger conditions, subagent table, HITL policy — no stage logic |
| **Specialist agent** | `.cursor/agents/specialists/<stage>.md` | One job: produces statement, rules, checklist, output format |
| **Orchestrator agent** | `.cursor/agents/workflows/<name>-workflow.md` | Stage order, gate rules, confidence dashboard, merged output |
| **Confidence rubric** | `.cursor/CONFIDENCE-SCORING.md` | HITL scoring for all agents |

## Built-in pipelines

| Pipeline | Orchestrator | Skill | Command |
|----------|--------------|-------|---------|
| LLD design round | `lld-design-round-workflow` | `lld-design-round` | `/lld-round` |
| Backend release | `backend-release-workflow` | `backend-release-pipeline` | `/backend-release` |
| Design & ship | `design-and-ship-workflow` | `design-and-ship` | `/design-and-ship` |
| Extend / scaffold | — | `agentic-workflows` | `/extend-workflow` |

## Agent authoring rules

1. **Specialist agents are narrow** — one stage, one output format, confidence one-liner on every report.
2. **Orchestrators delegate** — orchestrators do not redo specialist work; they pass prior stage summaries forward.
3. **Skills are routing only** — skills name which agent to invoke and under what trigger; they do not contain stage logic.
4. **Commands are thin** — point at the orchestrator plus constraints; no stage logic in commands.
5. **HITL is mandatory** — every agent follows `CONFIDENCE-SCORING.md`; orchestrators block approval/deploy on pending Required items.
6. **No conditional checklist items** — every item in a checklist is unconditional; remove items that only apply sometimes.

## Frontmatter requirements

### Specialist agent
```yaml
---
name: <stage-name>
description: <Specific trigger condition. What it produces. Use proactively when [concrete scenario].>
---
```

### Orchestrator agent
```yaml
---
name: <pipeline-name>-workflow
description: <What it orchestrates and when to use it.>
---
```

### SKILL.md
```yaml
---
name: <pipeline-name>
description: >-
  <Third-person WHAT + WHEN triggers — mention key terms users will say.>
disable-model-invocation: true
---
```

## Orchestrator Task tool prompt pattern

```text
Follow <specialist-name> in .cursor/agents/specialists/<file>.md.
# or for orchestrators:
Follow <workflow-name> in .cursor/agents/workflows/<file>.md.
Repo: {cwd}
Prior stages: {summary of prior stage output}
Report confidence per .cursor/CONFIDENCE-SCORING.md.
```

Use `readonly: true` for design-only stages. Use `shell` or `generalPurpose` when the agent must execute commands.

## New pipeline creation checklist

- [ ] Name the pipeline `kebab-case` (e.g. `api-migration`)
- [ ] Create specialist agents: `.cursor/agents/specialists/<stage>.md` for each stage
- [ ] Create orchestrator: `.cursor/agents/workflows/<name>-workflow.md` with stage order and gate rules
- [ ] Create skill: `.cursor/skills/<name>/SKILL.md` with trigger conditions and agent table
- [ ] Create command: `.cursor/commands/<name>.md` with problem placeholder and orchestrator name
- [ ] Add a row to the Built-in pipelines table above
- [ ] Confirm every new agent has a confidence one-liner and output format section
