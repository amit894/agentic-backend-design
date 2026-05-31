---
name: agentic-workflows
description: >-
  Guides creation and extension of Cursor agentic workflows—SKILL.md, subagents,
  commands, and orchestrators. Use when adding a new pipeline stage, scaffolding
  a workflow, or deciding between skill vs agent vs command.
disable-model-invocation: true
---

# Agentic Workflows (Skills + Agents)

Read [.cursor/WORKFLOW-ARCHITECTURE.md](../../WORKFLOW-ARCHITECTURE.md) for the full model. Use this skill when the user wants to **extend** or **create** workflows.

## SKILL.md vs agent vs command

| Use | Artifact | Why |
|-----|----------|-----|
| Teach main agent how to run pipelines | `skills/<name>/SKILL.md` | Discovery, sequencing, links, HITL policy |
| Isolated specialist behavior | `agents/<stage>.md` | Focused system prompt + output schema |
| User-facing shortcut | `commands/<name>.md` | `/slash` expands to orchestrator prompt |
| Multi-stage coordination | `agents/<name>-workflow.md` | Stage order, gates, consolidated report |

**Do not** put stage-by-stage logic only in SKILL.md — keep executable behavior in **agents**.

## When to create a new workflow

Create a **new pipeline** when:

- ≥2 ordered stages with different expertise or tools
- Stages have hard/soft gates (tests, HITL, deploy)
- The flow will be reused across sessions or teammates

Add a **single agent** only when:

- One new check (lint, contract test, threat model) plugs into an existing orchestrator

## Scaffold a new pipeline

1. **Specialist** — copy `.cursor/agents/backend-test.md` or `lld-requirements.md`; rename, rewrite checklist and output format.
2. **Orchestrator** — copy `backend-release-workflow.md` or `lld-design-round-workflow.md`; update stage table, gates, confidence dashboard.
3. **Skill** — create `.cursor/skills/<pipeline>/SKILL.md`:

```markdown
---
name: <pipeline>
description: >-
  <Third person WHAT + WHEN triggers. Mention backend, LLD, deploy, etc.>
disable-model-invocation: true
---

# <Title>

## Human-in-the-loop
Per [CONFIDENCE-SCORING.md](../../CONFIDENCE-SCORING.md).

## Subagents
| Agent | Stage |
|-------|-------|

## Quick start
\`\`\`
Use the <pipeline>-workflow subagent to ...
\`\`\`
```

4. **Command** — create `.cursor/commands/<pipeline>.md` (2–15 lines): problem scope, orchestrator name, HITL/deploy rules.
5. **Register** — add row to `WORKFLOW-ARCHITECTURE.md` built-in table.

## Plug into existing pipelines

| Extend | Action |
|--------|--------|
| LLD round | New `lld-<stage>.md` + insert stage in `lld-design-round-workflow.md` + update `lld-design-round/SKILL.md` |
| Release | New `backend-<stage>.md` + insert in `backend-release-workflow.md` + update `backend-release-pipeline/SKILL.md` |
| HITL only | No new agent — ensure output uses [CONFIDENCE-SCORING.md](../../CONFIDENCE-SCORING.md) |

## Invocation patterns

**Full pipeline:**
```
Use the <orchestrator-name> subagent. Scope: [module]. Target: [local|dry-run].
```

**With skill loaded:**
```
Follow agentic-workflows skill. Add a security-review stage to backend-release-workflow.
```

**Task tool (programmatic):**
```text
Follow <agent> in .cursor/agents/<file>.md. Repo: {cwd}. Prior: {prior summary}.
Report confidence per CONFIDENCE-SCORING.md.
```

## Existing skills in this repo

| Skill | Orchestrator |
|-------|--------------|
| [lld-design-round](../lld-design-round/SKILL.md) | `lld-design-round-workflow` |
| [backend-release-pipeline](../backend-release-pipeline/SKILL.md) | `backend-release-workflow` |

## Quality bar for new agents

- Description includes **when to delegate** (specific triggers)
- Output format is copy-pasteable markdown with tables
- Confidence section before `## Output format`
- Constraints: run commands yourself when applicable; no invented pass/fail

## Examples

**Add API contract test stage to release:**
1. Create `backend-contract-test.md`
2. Insert after `backend-test` in `backend-release-workflow.md` (hard gate optional)
3. Update `backend-release-pipeline/SKILL.md` pipeline order
4. Update `/backend-release` command stage list

**New interview-only workflow:**
1. Copy `lld-interviewer.md` pattern
2. Orchestrator with stages 1–6 only (no deploy)
3. Skill + `/mock-lld` command
