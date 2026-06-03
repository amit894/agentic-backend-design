---
name: lld-open-questions
description: LLD open questions resolver. Triages every open question from PROBLEM-BRIEF.md against the completed design stages, marks each Resolved/Assumption-made/Deferred, and surfaces any new questions the design introduced. Use as the final stage of a design round.
---

**Produces**: A resolution table for every open question from PROBLEM-BRIEF.md, a list of new questions surfaced during design, and a final design status verdict.

## Rules

- Every open question from PROBLEM-BRIEF.md is addressed — none are silently dropped.
- Resolution status is one of three: **Resolved** (design makes an explicit decision), **Assumption-made** (no definitive answer; state the assumption and its confidence), **Deferred** (genuinely out of scope; state why and when it should be revisited).
- Resolved questions cite the stage and section that resolves them (e.g. "Trade-offs §D3").
- New questions are questions the design stages introduced that were not in the brief — capture them so they are not lost.
- Deferred questions that touch a Must-priority FR become Required HITL.

## Checklist

- [ ] Every PROBLEM-BRIEF.md open question: resolution status + evidence + owner stage
- [ ] Assumption-made items: state the assumption explicitly, confidence %, and consequence if wrong
- [ ] Deferred items: state why deferred and the condition under which it should be re-opened
- [ ] New questions surfaced by design: list with suggested owner stage for resolution
- [ ] Final design status: COMPLETE / COMPLETE WITH ASSUMPTIONS / PENDING DECISIONS

> Confidence scoring: follow `.cursor/CONFIDENCE-SCORING.md`. Label every item with `Confidence %` | `Evidence (Verified / Inferred / Assumed)` | `HITL (Required / Recommended / Optional)`. End the report with **Overall confidence: NN%**, **HITL summary: N required / N recommended / N optional**, **Human review queue: one validation question per Required item**.

## Output

```markdown
# LLD Open Questions

## Resolution table

| # | Question (from PROBLEM-BRIEF.md) | Status | Decision / Assumption | Evidence | Stage that resolves |
|---|----------------------------------|--------|----------------------|----------|---------------------|
| 1 | [question text] | Resolved / Assumption-made / Deferred | [what was decided or assumed] | | |

## Assumptions log
| Assumption | Confidence % | Consequence if wrong | HITL |
|------------|--------------|---------------------|------|

## Deferred questions
| Question | Reason deferred | Re-open when |
|----------|----------------|--------------|

## New questions surfaced by design
| Question | Introduced by | Suggested owner stage |
|----------|--------------|----------------------|

## Final design status
COMPLETE | COMPLETE WITH ASSUMPTIONS (N) | PENDING DECISIONS (N)

### Resolution confidence
| Item | Confidence % | Evidence | HITL |
|------|--------------|----------|------|

**Overall confidence**: NN%
**HITL summary**: N required / N recommended / N optional
**Human review queue**:
- [ ] [validation question per Required item]
```
