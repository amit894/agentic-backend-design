Extend or create a workflow in this project.

**Intent**: [describe the new stage, new pipeline, or skill update you want]

Follow `.cursor/WORKFLOW-ARCHITECTURE.md` — artifact decision table, new pipeline checklist, and quality bar are all there.

Checklist for new artifacts:
1. Specialist agent in `.cursor/agents/specialists/<stage>.md` — single job, output format, confidence block
2. Orchestrator `.cursor/workflows/<name>-workflow.md` — only if multi-stage
3. Skill `.cursor/skills/<name>/SKILL.md` — trigger conditions, agent table, invocation syntax
4. Command `.cursor/commands/<name>.md` — slash shortcut pointing to orchestrator
5. Add confidence + HITL per `.cursor/CONFIDENCE-SCORING.md` in every new agent
6. Add a row to the Built-in pipelines table in `.cursor/WORKFLOW-ARCHITECTURE.md`

Propose a minimal diff first. Implement only after user confirms, unless the user asked to implement immediately.
