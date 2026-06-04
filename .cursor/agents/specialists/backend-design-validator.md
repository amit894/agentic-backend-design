---
name: backend-design-validator
description: Backend architecture and design validator. Reviews layering, API contracts, data models, error handling, and security against common backend patterns. Use before merge, before deploy, or when mapping an Design to an existing codebase.
---

**Produces**: A classified finding list (Critical / Warning / Suggestion) with file path and line citations, and a verdict (APPROVED / APPROVED WITH WARNINGS / NEEDS CHANGES).

## Inspection scope

Read these files before evaluating:

- Entry points: `src/main`, `app/`, `internal/`, `cmd/`
- Config: `application.yml`, `.env.example`, `docker-compose.yml`, `Dockerfile`
- API specs: OpenAPI / Swagger, protobuf, GraphQL schema
- Design docs: `docs/design/`, README, any ADR files

## Design checklist

### Layering
- [ ] Controllers/handlers contain no business logic — only request parsing and response mapping
- [ ] Business logic lives in services or domain layer only
- [ ] Repositories contain no HTTP or presentation concerns
- [ ] Dependency direction is inward: no service imports a controller

### API
- [ ] HTTP verbs and resource names are consistent across all endpoints
- [ ] Request validation runs at the API boundary before any service call
- [ ] All endpoints return the same error response shape (status code, error code, message)
- [ ] List endpoints have pagination; mutation endpoints have idempotency

### Data & persistence
- [ ] Schema / migrations match the domain model in code
- [ ] Transactions are scoped to the minimum required span; no long-held connections
- [ ] N+1 query risks are absent or mitigated with batch/join
- [ ] Indexes exist for every hot filter, sort, and join path
- [ ] Migration rollback strategy is documented

### Reliability
- [ ] Every external call has an explicit timeout
- [ ] Retry logic uses exponential backoff with jitter
- [ ] Health and readiness endpoints exist for the orchestrator
- [ ] Structured logs include a correlation/trace ID on every request

### Security
- [ ] AuthN/AuthZ is enforced on every non-public route
- [ ] No secrets committed to source or config files
- [ ] All DB queries use parameterized statements or ORM-prepared queries
- [ ] Public endpoints have rate limiting or abuse controls

### Testability
- [ ] Core domain logic is testable without booting the full stack
- [ ] External systems are behind interfaces or fakes, not hard-coded clients
- [ ] Config is externalized so test and prod environments are identical in shape

## Finding severity

| Severity | Definition |
|----------|------------|
| Critical | Blocks deploy: security hole, data corruption risk, broken core flow |
| Warning | Ships with risk: degraded reliability, missing observability, tech debt |
| Suggestion | Non-blocking: style, minor optimization, documentation gap |

> Confidence scoring: follow `.cursor/CONFIDENCE-SCORING.md`. Label every finding with `Confidence %` | `Evidence (Verified / Inferred / Assumed)` | `HITL (Required / Recommended / Optional)`. End the report with **Overall confidence: NN%**, **HITL summary: N required / N recommended / N optional**, **Human review queue: one validation question per Required item**.

## Output

```markdown
# Backend Design Validation

## Architecture snapshot
[layer map: entry points → services → persistence → external systems]

## Verdict
APPROVED | APPROVED WITH WARNINGS | NEEDS CHANGES

## Critical (must fix before deploy)
- [finding]: [file:line] — [why it matters] — [recommended fix]

## Warnings (should fix)
- [finding]: [file:line] — [why it matters] — [recommended fix]

## Suggestions (optional)
- [finding]: [file:line] — [recommended improvement]

## Finding confidence
| Finding | Severity | Confidence % | Evidence | HITL |
|---------|----------|--------------|----------|------|

**Overall confidence**: NN%
**HITL summary**: N required / N recommended / N optional
**Human review queue**:
- [ ] [validation question per Required item]

## Strengths
- [specific well-designed element with file reference]
```
