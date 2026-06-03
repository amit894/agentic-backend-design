---
name: lld-testing-strategy
description: LLD testing strategy designer. Produces a test matrix covering unit, integration, API, and performance layers with specific test cases for each functional requirement. Use after component sketch and trade-offs exist in a design round.
---

**Produces**: A test matrix (layer × scope), specific named test cases for every FR and critical failure path, test data requirements, and coverage targets — derived from the requirements and component sketch.

## Rules

- Every FR from the requirements stage has at least one named test case.
- Every failure path from the sequence flows stage has at least one test case.
- Test cases are named precisely: `get_existingKey_returnsValueAndPromotesToMRU`, not "test get".
- Layer boundaries are explicit: unit tests do not start the full stack; integration tests hit real boundaries.
- External systems in unit tests are replaced with fakes or in-memory implementations, not mocks of concrete classes.
- Performance/load tests have a measurable pass criterion tied to NFRs (e.g. "p99 < 1ms at 10k RPS").
- Test data setup is stated — seed scripts, fixtures, or builder patterns; no "assume data exists".

## Checklist

- [ ] Unit tests: one per domain method with edge cases (empty cache, at capacity, TTL expired, negative cache)
- [ ] Integration tests: DB round-trip, Redis round-trip, eviction under load — each hits a real dependency
- [ ] API / contract tests: every endpoint happy path + every documented error code
- [ ] Performance / benchmark tests: tied to p50/p95/p99 NFRs from requirements; run against local or CI environment
- [ ] Failure path tests: circuit breaker open, L2 unavailable, eviction race condition
- [ ] Test data: state what each test requires and how it is set up
- [ ] Coverage target: state minimum line/branch coverage and which modules are exempt and why

> Confidence scoring: follow `.cursor/CONFIDENCE-SCORING.md`. Label every claim with `Confidence %` | `Evidence (Verified / Inferred / Assumed)` | `HITL (Required / Recommended / Optional)`. End the report with **Overall confidence: NN%**, **HITL summary: N required / N recommended / N optional**, **Human review queue: one validation question per Required item**.

## Output

```markdown
# LLD Testing Strategy

## Test matrix
| Layer | Scope | Tool | Runs in |
|-------|-------|------|---------|
| Unit | Domain logic, eviction, TTL | JUnit 5 / pytest / go test | Local + CI |
| Integration | Redis, DB, message broker | Testcontainers / real deps | CI |
| API / contract | All endpoints | REST-assured / httptest | CI |
| Performance | p99 under NFR load | k6 / JMH / wrk | CI nightly |
| Failure | Circuit breaker, L2 down | JUnit + WireMock | CI |

## Test cases

### Unit — [Component]
| Test name | Input | Expected output | Covers |
|-----------|-------|----------------|--------|
| `get_existingKey_returnsValueAndPromotesToMRU` | key present in shard | value returned; node moved to head | FR-1, FR-6 |

### Integration — [Boundary]
| Test name | Setup | Assertion |
|-----------|-------|-----------|

### API / contract — [Endpoint]
| Endpoint | Scenario | Expected status | Body assertion |
|----------|---------|----------------|---------------|

### Performance
| Scenario | Load | Pass criterion |
|----------|------|---------------|
| L1 read at peak | 10k concurrent | p99 < 1ms |

### Failure paths
| Scenario | Injection | Expected behaviour |
|----------|-----------|-------------------|

## Test data
- [what each layer requires and how it is seeded]

## Coverage target
- Minimum: NN% line, NN% branch
- Exempt: [generated code, config classes — reason]

## Testing confidence
| Layer | Confidence % | Evidence | HITL |
|-------|--------------|----------|------|

**Overall confidence**: NN%
**HITL summary**: N required / N recommended / N optional
**Human review queue**:
- [ ] [validation question per Required item]
```
