---
name: agentic-workflows
description: >-
  Guides creation and extension of workflows — SKILL.md, subagents, commands,
  and orchestrators. Use when adding a new pipeline stage, scaffolding a new
  workflow, or deciding between skill vs agent vs command.
disable-model-invocation: true
---

# Agentic Workflows

Reference: `.cursor/WORKFLOW-ARCHITECTURE.md` — layer model, artifact roles, built-in pipelines.

## Artifact decision table

| Need | Create |
|------|--------|
| One new check (lint, contract test, threat model) | Specialist agent only |
| New multi-step pipeline (≥2 ordered stages with gates) | Orchestrator + specialist agents + skill + command |
| Document how to invoke an existing pipeline | Update or add `SKILL.md` only |
| Faster slash-command invocation | `commands/<name>.md` only |

## New pipeline checklist

- [ ] **Specialist agents** — one per stage in `.cursor/agents/specialists/<stage>.md`
  - Required sections: produces statement, rules, checklist, confidence one-liner, output format
  - Copy pattern from `specialists/lld-requirements.md` or `specialists/backend-test.md`
- [ ] **Orchestrator** — `.cursor/agents/workflows/<name>-workflow.md`
  - Required sections: produces statement, pipeline diagram, stage invocation table, gate rules, output
  - Copy pattern from `workflows/lld-design-round-workflow.md` or `workflows/backend-release-workflow.md`
- [ ] **Skill** — `.cursor/skills/<name>/SKILL.md`
  - Required sections: trigger conditions, subagent table, invocation syntax, HITL policy
  - Frontmatter: `name`, `description` (third-person WHAT + WHEN), `disable-model-invocation: true`
- [ ] **Command** — `.cursor/commands/<name>.md`
  - Required: problem/intent placeholder, orchestrator name, deploy target or scope, HITL rule
- [ ] **Register** — add a row to the Built-in pipelines table in `.cursor/WORKFLOW-ARCHITECTURE.md`

## Extending an existing pipeline

| Pipeline | Insert |
|----------|--------|
| LLD round | New `lld-<stage>.md` → add to `lld-design-round-workflow.md` stage table → update `lld-design-round/SKILL.md` subagent table |
| Backend release | New `backend-<stage>.md` → add to `backend-release-workflow.md` stage table → update `backend-release-pipeline/SKILL.md` subagent table |

## Quality bar for every new agent

- Frontmatter `description` states the specific trigger condition.
- First line after frontmatter is a `**Produces**:` statement.
- Output format section produces copy-pasteable markdown with tables.
- Confidence one-liner is the last item before `## Output`.
- No "when applicable", "where needed", or "if relevant" — every checklist item is always required or not in the checklist at all.
