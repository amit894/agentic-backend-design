# Problem Brief

Fill this in before running `/lld-round`. The LLD workflow reads this as the source of truth for requirements, API design, and trade-offs.

---

## Title

[System name — e.g. "LRU Cache", "Real-time Notification Service", "URL Shortener"]

## One-line summary

[What the system does in one sentence — subject + verb + object]

## Background

[Why this exists: user pain, business constraint, or interview prompt. 2–4 sentences.]

## Users & actors

| Actor | Goal |
|-------|------|
| [e.g. End user] | [e.g. Retrieve a cached value in O(1)] |

## Core scenarios (happy path)

1. [First scenario — one sentence]
2. [Second scenario]
3. [Third scenario]

## Constraints

- **Scale**: [e.g. 1M DAU, 500 req/s peak, 10k documents, cache capacity = 1000 entries]
- **Latency**: [e.g. p95 < 200ms for reads, p95 < 500ms for writes]
- **Consistency**: [strong / eventual / per-entity — state which]
- **Budget / infra**: [cloud provider, on-prem, serverless allowed, cost ceiling]
- **Team / timeline**: [e.g. MVP in 2 weeks, solo dev, interview context]

## Known integrations

- [External API, auth provider, payment gateway, LLM provider, message broker — or "none"]

## Explicit non-goals (out of scope for MVP)

- [Named exclusion — e.g. "No TTL / time-based expiry"]
- [Named exclusion — e.g. "No distributed / multi-node support"]

## Open questions for the design round

- [Question that requires interviewer / stakeholder input]
- [Question that will become an assumption if unanswered]

## Interview context

- **Company / round type**: [e.g. backend LLD, 45 min, system design, Faang-style]
- **Depth expected**: [e.g. API + DB deep dive, scale discussion, code sketch]
