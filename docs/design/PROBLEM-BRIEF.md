# Problem Brief

Fill this in before running `/lld-round` or `/design-and-ship`. The LLD workflow reads this as the source of truth for requirements, API design, and trade-offs.

---

## Title

[System name — e.g. "LRU Cache", "URL Shortener", "Rate Limiter"]

## Output folder

`docs/design/problems/<problem-name>`

## One-line summary

[What the system does in one sentence — subject + verb + object]

## Background

[Why this exists: user pain, business constraint, or interview prompt. 2–4 sentences.]

## Users & actors

| Actor | Goal |
|-------|------|
| [e.g. End user] | [e.g. Resolve a short URL in < 5ms] |

## Core scenarios (happy path)

1. [First scenario — one sentence]
2. [Second scenario]
3. [Third scenario]

## Constraints

- **Scale**: [e.g. 100k RPS, 10B records, 5M hot entries in cache]
- **Latency**: [e.g. p99 < 5ms for reads, p99 < 100ms for writes]
- **Consistency**: [strong / eventual / per-entity — state which]
- **Concurrency**: [e.g. 256 shards, async write buffer depth]
- **Budget / infra**: [cloud provider, managed services allowed, cost ceiling]
- **Team / timeline**: [e.g. platform team, MVP in 2 weeks]

## Known integrations

- [External API, auth provider, message broker, LLM provider — or "none"]

## Explicit non-goals (out of scope for MVP)

- [Named exclusion]
- [Named exclusion]

## Open questions for the design round

- [Question that requires interviewer / stakeholder input]
- [Question that will become an assumption if unanswered]

## Interview context

- **Round type**: [e.g. Senior / Principal backend LLD, 60 min, FAANG-style]
- **Depth expected**: [e.g. API design, data structure internals, concurrency model, code sketch]
- **Key trade-offs to articulate**:
  - [Trade-off 1]
  - [Trade-off 2]
