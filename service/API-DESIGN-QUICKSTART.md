# Backend Job Service — API Design Quick Start

**Date**: 2026-06-04  
**Status**: Draft (HITL Approval Pending)  
**Overall Confidence**: 92%

---

## Two Core Endpoints

### 1. POST /jobs — Enqueue Job

```http
POST /api/v1/jobs
Content-Type: application/json

{
  "type": "email",
  "payload": "{\"recipient\": \"user@example.com\"}",
  "idempotency_key": "optional-uuid"
}
```

**Success (201 Created)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "type": "email",
  "status": "QUEUED",
  "created_at": "2026-06-04T12:00:00Z",
  "shard_id": 42
}
```

**Errors**:
- `400 INVALID_JOB_TYPE` — unknown type
- `400 INVALID_PAYLOAD` — missing, invalid, or > 1MB
- `409 IDEMPOTENCY_KEY_CONFLICT` — key exists; returns `existing_job_id`
- `503 SERVICE_UNAVAILABLE` — DB unreachable

**Latency**: p99 < 100ms  
**Throughput**: 10k jobs/sec (target)  
**Confidence**: 92%

---

### 2. GET /jobs/{jobId} — Get Job Status

```http
GET /api/v1/jobs/550e8400-e29b-41d4-a716-446655440000
```

**Success (200 OK)**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "type": "email",
  "payload": "{\"recipient\": \"user@example.com\"}",
  "status": "RUNNING",
  "attempt_count": 0,
  "max_attempts": 3,
  "created_at": "2026-06-04T12:00:00Z",
  "started_at": "2026-06-04T12:00:01Z",
  "completed_at": null,
  "next_retry_at": null,
  "error_code": null,
  "error_message": null,
  "worker_id": "worker-001"
}
```

**Errors**:
- `404 JOB_NOT_FOUND` — job doesn't exist
- `400 INVALID_JOB_ID` — not valid UUID

**Latency**: p99 < 100ms  
**Caching**: 30s TTL (eventual consistency)  
**Confidence**: 93%

---

## Key Design Decisions

### Idempotency: 409 Response (Not 200)

**Scenario**: Client enqueues same job twice with same idempotency_key

```
Request 1: POST /jobs { idempotency_key: "key-123", ... }
Response:  201 Created { id: "job-abc", ... }

Request 2: POST /jobs { idempotency_key: "key-123", ... }
Response:  409 Conflict { existing_job_id: "job-abc", ... }
```

**Why 409?**
- Signals "state conflict" to client
- Client knows to use `existing_job_id`
- Distinguishes from 201 (new) and 200 (generic)
- Prevents accidental duplicate creation

**Status**: HITL approval required

---

### Latency Breakdown (Both Endpoints < 100ms)

**POST /jobs**:
```
Validation:         < 1ms
Idempotency check:  < 5ms
UUID generation:    < 0.1ms
DB INSERT + WAL:    < 20ms
Serialization:      < 5ms
─────────────────────────
Total:              < 100ms ✓
```

**GET /jobs/{id}**:
```
UUID validation:    < 1ms
DB SELECT + index:  < 10ms
Serialization:      < 5ms
Network:            < 5ms
─────────────────────────
Total:              < 100ms ✓
```

---

### Throughput Analysis (10k/sec Target)

**Single PostgreSQL Capacity**: ~2.5k jobs/sec  
**Target**: 10k jobs/sec  
**Gap**: 4x shortfall

**Mitigations**:
1. In-process queue buffer (v1)
2. Batch INSERT (100 jobs/txn)
3. Write sharding by shard_id
4. Kafka (v2+)

**Status**: Load test required to validate actual throughput

---

## Error Codes Reference

| HTTP Status | Code | Endpoint | Meaning |
|-------------|------|----------|---------|
| 201 | — | POST /jobs | Job created successfully |
| 200 | — | GET /jobs/{id} | Job found; full state returned |
| 400 | INVALID_JOB_TYPE | POST /jobs | Unknown type |
| 400 | INVALID_PAYLOAD | POST /jobs | Payload missing, invalid, or > 1MB |
| 400 | INVALID_JOB_ID | GET /jobs/{id} | Job ID not valid UUID format |
| 404 | JOB_NOT_FOUND | GET /jobs/{id} | Job doesn't exist |
| 409 | IDEMPOTENCY_KEY_CONFLICT | POST /jobs | Key exists; returns existing_job_id |
| 503 | SERVICE_UNAVAILABLE | POST /jobs | DB unreachable or queue capacity exceeded |
| 500 | INTERNAL_ERROR | Any | Unexpected server error |

---

## Job Status Values

| Status | Meaning | Typical Transitions |
|--------|---------|-------------------|
| QUEUED | Waiting for worker pickup | → RUNNING |
| RUNNING | Worker executing external API | → COMPLETED, FAILED, UNAVAILABLE |
| COMPLETED | Success; job finished | Terminal |
| FAILED | Max retries exhausted | Terminal |
| CANCELLED | Job cancelled before execution | Terminal |
| UNAVAILABLE | Circuit breaker open | → QUEUED (after 30s reset) |

---

## Critical HITL Approval Items

### 1. Idempotency Response Code (Confidence 91%)
- [ ] Decision: 409 Conflict or 200 OK?
- [ ] Recommendation: 409 for semantic clarity
- [ ] Approved: _______________

### 2. Enqueue Throughput (Confidence 82%)
- [ ] Load test baseline needed (10k/sec target)
- [ ] Single DB may bottleneck at 2.5k/sec
- [ ] Needs: write sharding or Kafka v2?
- [ ] Approved: _______________

### 3. Idempotency Key Retention (Confidence 80%)
- [ ] Policy: Infinite, 30-day TTL, or manual?
- [ ] Recommendation: 30-day TTL
- [ ] Approved: _______________

---

## Database Tables (Required)

```sql
CREATE TABLE jobs (
  id UUID PRIMARY KEY,
  type VARCHAR(50) NOT NULL,
  payload JSONB NOT NULL,
  status VARCHAR(20) NOT NULL,
  shard_id INT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  next_retry_at TIMESTAMP,
  error_code INT,
  error_message TEXT,
  worker_id VARCHAR(100),
  idempotency_key VARCHAR(256),
  attempt_count INT DEFAULT 0,
  max_attempts INT DEFAULT 3
);

CREATE UNIQUE INDEX idx_jobs_idempotency_key 
  ON jobs(idempotency_key) WHERE idempotency_key IS NOT NULL;
CREATE INDEX idx_jobs_id ON jobs(id);
CREATE INDEX idx_jobs_shard_status_created 
  ON jobs(shard_id, status, created_at);

CREATE TABLE jobs_dlq (
  id UUID PRIMARY KEY,
  type VARCHAR(50) NOT NULL,
  payload JSONB NOT NULL,
  final_attempt_count INT,
  error_code INT,
  error_message TEXT,
  moved_at TIMESTAMP DEFAULT NOW()
);
```

---

## Next Steps

1. **HITL Review** (2026-06-11) — Critical item approvals
2. **Load Testing** (2026-06-11 to 2026-06-15) — Validate 10k/sec throughput
3. **Implementation** (2026-06-18) — Code POST /jobs and GET /jobs/{id}
4. **Testing** (2026-06-25) — Full integration validation
5. **Production Readiness** (2026-07-02) — Monitoring and docs

---

**Overall Confidence**: 92%  
**Status**: Draft → Pending HITL Sign-Off

