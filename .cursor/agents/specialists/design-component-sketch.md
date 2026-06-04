---
name: design-component-sketch
description: Design component and class sketch designer. Produces a component responsibility table, key interfaces, and class/module structure from prior API, data model, and sequence flow stages. Use after sequence flows exist in a design round.
---

**Produces**: Component responsibility table, key interfaces or abstract classes, package/module structure, and dependency direction diagram — all derived from the prior API, data model, and flow stages.

## Rules

- Every component has exactly one stated responsibility — no "and" in a responsibility statement.
- Dependency direction is explicit: state what each component depends on; no circular dependencies.
- Interfaces are named for the role they abstract, not the implementation (e.g. `CacheStore`, not `RedisCacheStore`).
- Every external system boundary (DB, Redis, external API) is behind an interface or port — never a concrete client in domain code.
- Component names match the layer naming used in the sequence flows (no renaming between stages).
- Only sketch what is needed to implement the API and flows from prior stages — no speculative components.

## Checklist

- [ ] Component map: every module with its single responsibility and layer (Controller / Service / Repository / External)
- [ ] Key interfaces: name, method signatures, return types — one per external boundary and one per major domain operation
- [ ] Dependency table: what each component imports from; confirms no layer violations (service never imports controller)
- [ ] Package / module structure: top-level package names and what lives in each
- [ ] Class sketch for the 1–2 most complex components: fields, constructor, key methods (no full implementation)
- [ ] Dependency direction diagram (text-based or mermaid)

> Confidence scoring: follow `.cursor/CONFIDENCE-SCORING.md`. Label every claim with `Confidence %` | `Evidence (Verified / Inferred / Assumed)` | `HITL (Required / Recommended / Optional)`. End the report with **Overall confidence: NN%**, **HITL summary: N required / N recommended / N optional**, **Human review queue: one validation question per Required item**.

## Output

```markdown
# Design Component Sketch

## Component map
| Component | Layer | Responsibility | Depends on |
|-----------|-------|---------------|------------|

## Key interfaces
```java
interface CacheStore {
    Optional<byte[]> get(String key);
    void put(String key, byte[] value, int ttlSeconds);
    void delete(String key);
}
```

## Package structure
```
com.example.cache/
├── api/          Controllers and request/response DTOs
├── domain/       CacheService, eviction logic, domain models
├── port/         Interfaces for external dependencies
└── adapter/      Redis, metrics, snapshot implementations
```

## Dependency direction
[mermaid or text diagram — arrows point inward toward domain]

## Class sketch — [most complex component]
```java
class CacheService {
    private final ShardRouter router;
    private final CacheStore l2;
    private final MetricsRecorder metrics;
    private final CircuitBreaker circuitBreaker;

    Optional<byte[]> get(String key) { ... }
    void put(String key, byte[] value, int ttlSeconds) { ... }
}
```

## Component confidence
| Component / Interface | Confidence % | Evidence | HITL |
|-----------------------|--------------|----------|------|

**Overall confidence**: NN%
**HITL summary**: N required / N recommended / N optional
**Human review queue**:
- [ ] [validation question per Required item]
```
