# Confidence Scoring — Human-in-the-Loop (HITL)

All workflow agents and orchestrators use this rubric. Scores guide **what a human must validate** before trusting or shipping output.

## Score scale (0–100%)

| Range | Label | Meaning |
|-------|-------|---------|
| **90–100** | High | Verified in repo, test run, command output, or official docs |
| **70–89** | Medium | Strong inference from code/config; not fully executed or proven |
| **50–69** | Low | Partial context; reasonable assumption stated explicitly |
| **&lt;50** | Very low | Speculative; missing data; needs human input before use |

## Evidence types

| Type | When to use |
|------|-------------|
| **Verified** | You ran a command, read the file, or saw a test/deploy result |
| **Inferred** | Logical conclusion from structure, naming, or partial reads |
| **Assumed** | Industry default, interview norm, or user brief not confirmed in code |

## HITL action (per item and per stage)

| HITL | Trigger | Human action |
|------|---------|--------------|
| **Required** | Confidence &lt;70%, OR **Assumed** on a Must/FR/Critical item, OR contradicts prior stage | Explicit approve / reject / edit before proceed |
| **Recommended** | 70–89%, or Inferred on security/scale/cost decisions | Spot-check; confirm or override |
| **Optional** | ≥90% and Verified | Audit sample only |

## Stage rollup (orchestrators)

- **Stage confidence** = minimum confidence among **Required** HITL items in that stage; if none Required, use weighted average of top 5 claims (weight: Critical 2×, Warning 1×).
- **Pipeline confidence** = minimum stage confidence across completed stages.
- **Proceed rule**: Do not auto-advance to deploy (or mark design **Approved**) if any stage has **Required** HITL items still **Pending**.

## Output conventions

Every specialist report must include:

1. **Per finding row**: `Confidence %` | `Evidence` | `HITL`
2. **Stage overall**: `Overall confidence: NN%` | `HITL summary: N required, N recommended`
3. **Human review queue**: bullet list of all **Required** items with one-line validation question

Orchestrators add a **Confidence dashboard** table across stages.
