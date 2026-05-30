# LLD Interview Rubric (Backend / LLM Features)

Use for self-assessment or with the `lld-interviewer` subagent.

## Scoring scale

| Score | Meaning |
|-------|---------|
| 1 | Missing or incorrect |
| 2 | Partial; major gaps |
| 3 | Solid; minor gaps |
| 4 | Strong; hire-level depth |

## Dimensions

### 1. Requirements & scope (weight: 15%)
- Clarifies FR/NFR before designing
- States assumptions and out-of-scope explicitly
- Handles scale, privacy, and failure expectations

### 2. API design (weight: 20%)
- RESTful, consistent naming and error model
- Validation, pagination, idempotency where needed
- Chat/upload endpoints separated; citations in responses (LLM systems)

### 3. Data model (weight: 20%)
- Entities match API and flows
- Indexes for hot queries; blob vs metadata split
- Pipeline status and chunk/vector storage (RAG systems)

### 4. Flows & reliability (weight: 20%)
- Sequence diagrams for critical paths
- Transaction boundaries, retries, partial failure
- LLM fallback when provider unavailable

### 5. Trade-offs (weight: 15%)
- Compares ≥2 options for major decisions
- Links choices to NFRs (latency, cost, complexity)

### 6. Communication (weight: 10%)
- Structured progression; checks interviewer understanding
- Admits unknowns; prioritizes MVP vs extensions

## Hire bar (guidance)

| Total (avg) | Signal |
|-------------|--------|
| ≥ 3.5 | Strong hire for LLD round |
| 3.0 – 3.4 | Hire with coaching |
| 2.5 – 2.9 | Borderline |
| < 2.5 | No hire |

## Red flags
- Jumps to code/classes before requirements
- No error or edge-case handling
- Single design option with no rationale
- Ignores grounding/citations in doc-QA systems
- Cannot explain how to test without production LLM

## Green flags
- Quantifies scale ("10k docs, 100 QPS chat")
- Explicit observability day one
- Clear MVP vs phase 2
- Maps design to test and deploy path
