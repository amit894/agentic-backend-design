# Agentic Workflow Architecture

How **skills**, **agents**, **commands**, and **orchestrators** work together in this repo.

## Layer model

```
User
  │
  ├─► .cursor/commands/*.md        Slash shortcuts (/lld-round, /backend-release)
  │
  ├─► .cursor/skills/*/SKILL.md      Meta-instructions for the main agent (when to run what)
  │
  └─► .cursor/agents/*.md            Specialist prompts (single stage or orchestrator)
        │
        └─► Task tool / "Use the X subagent"  Isolated execution per stage
```

| Artifact | Path | Role |
|----------|------|------|
| **Command** | `.cursor/commands/<name>.md` | Expands to a full prompt; best for repeat user actions |
| **Skill** | `.cursor/skills/<name>/SKILL.md` | Teaches discovery, sequencing, HITL, and links to agents; use for multi-file workflows |
| **Specialist agent** | `.cursor/agents/<stage>.md` | One job, one output format, confidence scores |
| **Orchestrator agent** | `.cursor/agents/*-workflow.md` | Runs stages in order, merges reports, enforces gates |
| **Rubric** | `.cursor/CONFIDENCE-SCORING.md` | HITL confidence for all stages |

## When to add what

| Need | Add |
|------|-----|
| New single concern (e.g. security scan) | Specialist agent only |
| New multi-step pipeline | Orchestrator + specialist agents + skill + command |
| Document how to use existing pipelines | Update or add `SKILL.md` only |
| Faster invocation | `commands/<name>.md` only |
| Cross-cutting rule always on | `.cursor/rules/*.mdc` (not covered here) |

## Built-in pipelines

| Pipeline | Orchestrator | Skill | Command |
|----------|--------------|-------|---------|
| LLD design round | `lld-design-round-workflow` | `lld-design-round` | `/lld-round` |
| Backend release | `backend-release-workflow` | `backend-release-pipeline` | `/backend-release` |
| Extend / scaffold | — | `agentic-workflows` | `/extend-workflow` |

## Agentic workflow rules

1. **Specialists stay narrow** — one stage, one output template, confidence + HITL on every report.
2. **Orchestrators delegate** — do not redo specialist work; pass prior stage summaries forward.
3. **Skills do not replace agents** — skills tell the main agent *which* agent to invoke and *in what order*.
4. **Commands stay thin** — point at orchestrator + constraints; keep stage logic in agents.
5. **HITL is mandatory** — follow [CONFIDENCE-SCORING.md](./CONFIDENCE-SCORING.md); block ship/approve on pending Required items.
6. **New problem statements** — update `docs/design/PROBLEM-BRIEF.md` (kit) or problem section in LLD; re-run `/lld-round`.

## Creating a new workflow (checklist)

- [ ] Name pipeline `kebab-case` (e.g. `api-migration`)
- [ ] Add specialist agent(s) under `.cursor/agents/<stage>.md`
- [ ] Add orchestrator `.cursor/agents/<name>-workflow.md` with stage order and gates
- [ ] Add `.cursor/skills/<name>/SKILL.md` with description, triggers, agent table, HITL note
- [ ] Add `.cursor/commands/<name>.md` for slash invocation
- [ ] Wire confidence blocks (copy from an existing agent)
- [ ] Update this file's "Built-in pipelines" table
- [ ] If design-related, extend `docs/design/LLD-TEMPLATE.md` section 11 HITL log as needed

## SKILL.md frontmatter (required)

```yaml
---
name: my-workflow
description: >-
  Third-person description: WHAT it does and WHEN to use it (trigger terms).
disable-model-invocation: true
---
```

Omit `disable-model-invocation` only if the skill should auto-load from ambient context.

## Specialist agent frontmatter (required)

```yaml
---
name: my-stage
description: Specific trigger. Use proactively when [concrete scenario].
---
```

Body: When invoked → checklist → constraints → confidence section → output format.

## Orchestrator delegation (Task tool)

```text
Follow <agent-name> in .cursor/agents/<file>.md.
Repo: {cwd}
Prior stages: {summary}
Include Overall confidence, HITL summary, Human review queue per CONFIDENCE-SCORING.md.
```

Use `readonly: true` for design-only stages; `shell` or `generalPurpose` when commands must run.

## Optional: application-level agentic (runtime)

This repo's **Spring Boot + Spring AI tools** (`DocumentAgentTools`, `ChatUseCaseService`) are separate from Cursor agents. Cursor workflows design and validate that code; they do not replace in-app tool calling.
