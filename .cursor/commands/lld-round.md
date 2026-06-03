Run a full LLD design round.

**Problem**: [state the problem here, or point to `docs/design/PROBLEM-BRIEF.md`]

Use the `lld-design-round-workflow` subagent. Merge stage outputs into `docs/design/problems/<problem-name>/lld.md` (create the folder if it does not exist) using the `docs/design/LLD-TEMPLATE.md` section structure.

Include a confidence dashboard and consolidated human review queue per `.cursor/CONFIDENCE-SCORING.md`.

If this repo already has code for the system being designed, run stage 7 (`backend-design-validator`) to produce a design-vs-implementation gap list.

Do not write production code unless the user asks. Design deliverable only.
