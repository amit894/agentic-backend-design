# Agentic Workflow Architecture

## Layer model

```
User
  │
  ├─► .cursor/commands/*.md          Slash shortcuts (/lld-round, /backend-release)
  │
  ├─► .cursor/skills/*/SKILL.md      Routing: trigger conditions, agent table, invocation syntax
  │
  ├─► .cursor/agents/specialists/*.md   Specialists — one job, one output format
  │
  └─► .cursor/workflows/*.md            Orchestrators — stage order, gates, merged output
        │
        └─► Task tool / subagent invocation
```

## Artifact roles

| Artifact | Path | Responsibility |
|----------|------|---------------|
| **Command** | `.cursor/commands/<name>.md` | Problem/intent placeholder + orchestrator name + constraints |
| **Skill** | `.cursor/skills/<name>/SKILL.md` | Trigger conditions, subagent table, HITL policy — no stage logic |
| **Specialist agent** | `.cursor/agents/specialists/<stage>.md` | One job: produces statement, rules, checklist, output format |
| **Orchestrator agent** | `.cursor/workflows/<name>-workflow.md` | Stage order, gate rules, confidence dashboard, merged output |
| **Confidence rubric** | `.cursor/CONFIDENCE-SCORING.md` | HITL scoring for all agents |

## Built-in pipelines

| Pipeline | Orchestrator | Skill | Command |
|----------|--------------|-------|---------|
| LLD design round | `lld-design-round-workflow` | `lld-design-round` | `/lld-round` |
| Backend release | `backend-release-workflow` | `backend-release-pipeline` | `/backend-release` |
| Design & ship | `design-and-ship-workflow` | `design-and-ship` | `/design-and-ship` |

## When to create what

| Need | Create |
|------|--------|
| One new check (lint, contract test, threat model) | Specialist agent in `agents/specialists/` only |
| New multi-step pipeline (≥2 ordered stages with gates) | Orchestrator + specialist agents + skill + command |
| Document how to invoke an existing pipeline | Update or add `skills/<name>/SKILL.md` only |
| Faster slash-command invocation | `commands/<name>.md` only |

## Extending an existing pipeline

| Pipeline | Steps |
|----------|-------|
| LLD round | New `agents/specialists/lld-<stage>.md` → add row to `.cursor/workflows/lld-design-round-workflow.md` stage table → update `skills/lld-design-round/SKILL.md` subagent table |
| Backend release | New `agents/specialists/backend-<stage>.md` → add row to `.cursor/workflows/backend-release-workflow.md` stage table → update `skills/backend-release-pipeline/SKILL.md` subagent table |

## Agent authoring rules

1. **Specialist agents are narrow** — one stage, one output format, confidence one-liner on every report.
2. **Orchestrators delegate** — orchestrators do not redo specialist work; they pass prior stage summaries forward.
3. **Skills are routing only** — skills name which agent to invoke and under what trigger; they do not contain stage logic.
4. **Commands are thin** — point at the orchestrator plus constraints; no stage logic in commands.
5. **HITL is mandatory** — every agent follows `CONFIDENCE-SCORING.md`; orchestrators block approval/deploy on pending Required items.
6. **No conditional checklist items** — every item in a checklist is unconditional; remove items that only apply sometimes.

## Quality bar for every new agent

- Frontmatter `description` states the specific trigger condition.
- First line after frontmatter is a `**Produces**:` statement.
- Output format section produces copy-pasteable markdown with tables.
- Confidence one-liner is the last item before `## Output`.
- No "when applicable", "where needed", or "if relevant" — every checklist item is always required or not in the checklist at all.

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
Follow <workflow-name> in .cursor/workflows/<file>.md.
Repo: {cwd}
Prior stages: {summary of prior stage output}
Report confidence per .cursor/CONFIDENCE-SCORING.md.
```

Use `readonly: true` for design-only stages. Use `shell` or `generalPurpose` when the agent must execute commands.

## New pipeline creation checklist

- [ ] Name the pipeline `kebab-case` (e.g. `api-migration`)
- [ ] Create specialist agents: `.cursor/agents/specialists/<stage>.md` for each stage
- [ ] Create orchestrator: `.cursor/workflows/<name>-workflow.md` with stage order and gate rules
- [ ] Create skill: `.cursor/skills/<name>/SKILL.md` with trigger conditions and agent table
- [ ] Create command: `.cursor/commands/<name>.md` with problem placeholder and orchestrator name
- [ ] Add a row to the Built-in pipelines table above
- [ ] Confirm every new agent has a confidence one-liner and output format section
