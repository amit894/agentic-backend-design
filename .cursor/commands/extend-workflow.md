Extend or create a Cursor agentic workflow in this project.

Follow the **agentic-workflows** skill (`.cursor/skills/agentic-workflows/SKILL.md`) and [.cursor/WORKFLOW-ARCHITECTURE.md](../WORKFLOW-ARCHITECTURE.md).

**User intent**: [describe new stage, new pipeline, or skill-only doc update]

If creating new artifacts:
1. Specialist agent(s) in `.cursor/agents/`
2. Orchestrator `*-workflow.md` if multi-stage
3. `SKILL.md` under `.cursor/skills/<name>/`
4. Slash command under `.cursor/commands/` if user-facing
5. Confidence + HITL per `CONFIDENCE-SCORING.md`
6. Update `WORKFLOW-ARCHITECTURE.md` built-in pipelines table

Propose a minimal diff first; implement after user confirms unless they asked to implement now.
