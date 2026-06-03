---
name: lld-observability
description: LLD observability and ops designer. Produces metrics, structured log fields, alert thresholds, health checks, and runbook pointers — derived from NFRs and the completed design. Use after trade-offs exist in a design round.
---

**Produces**: A complete observability plan — Prometheus metric names and labels, structured log schema, alert rules with thresholds tied to NFRs, health/readiness endpoint spec, and SLO definition.

## Rules

- Every metric name follows `<system>_<noun>_<unit>_total/seconds/bytes` convention (Prometheus naming).
- Every metric has a stated label set — no unbounded-cardinality labels (no user ID, no full URL).
- Every alert threshold is derived from an NFR — not invented. If no NFR exists for a signal, state the assumption.
- Structured log fields are named in `snake_case`; every request log includes `trace_id`, `span_id`, `latency_ms`, and `status`.
- Health endpoint returns a structured JSON body distinguishing `liveness` (process alive) from `readiness` (dependencies reachable).
- SLO is stated as: error budget = (1 − availability target) × window. Burn rate alert is at 2× and 5× normal consumption.
- On-call runbooks are named and scoped — one per alert class, not one per alert instance.

## Checklist

- [ ] Metrics: hit rate, miss rate, eviction rate, latency histograms (p50/p95/p99), L2 error rate, circuit breaker state
- [ ] Logs: request log schema, eviction audit log, circuit breaker state change log — fields and log level per event
- [ ] Alerts: hit rate floor, eviction rate ceiling, L2 error rate spike, p99 latency breach — threshold + severity + runbook
- [ ] Health endpoint: path, liveness check, readiness checks per dependency, response schema
- [ ] SLO: availability target, error budget, burn rate alert conditions
- [ ] Dashboard: key panels (hit rate trend, eviction rate, p99 latency, L2 circuit state) — Grafana or equivalent
- [ ] Runbooks: one per alert class with diagnosis steps and remediation actions

> Confidence scoring: follow `.cursor/CONFIDENCE-SCORING.md`. Label every claim with `Confidence %` | `Evidence (Verified / Inferred / Assumed)` | `HITL (Required / Recommended / Optional)`. End the report with **Overall confidence: NN%**, **HITL summary: N required / N recommended / N optional**, **Human review queue: one validation question per Required item**.

## Output

```markdown
# LLD Observability & Ops

## Metrics
| Metric name | Type | Labels | Purpose | Alert? |
|------------|------|--------|---------|--------|
| `cache_hits_total` | Counter | `shard`, `tier` | Hit rate numerator | No |
| `cache_misses_total` | Counter | `shard`, `tier` | Miss rate numerator | Yes — if hit rate < threshold |
| `cache_evictions_total` | Counter | `shard` | Eviction rate | Yes — if rate spikes |
| `cache_latency_seconds` | Histogram | `operation`, `tier` | p50/p95/p99 latency | Yes — p99 breach |
| `cache_circuit_breaker_state` | Gauge | `tier` | 0=closed, 1=open, 2=half-open | Yes — if open |

## Structured log schema
| Event | Level | Required fields |
|-------|-------|----------------|
| Cache request | DEBUG | `trace_id`, `key_hash`, `tier`, `hit`, `latency_ms` |
| Eviction | INFO | `shard_id`, `key_hash`, `reason`, `eviction_batch_size` |
| Circuit state change | WARN | `tier`, `old_state`, `new_state`, `failure_count` |

## Alerts
| Alert | Condition | Severity | Runbook |
|-------|-----------|----------|---------|
| HitRateLow | hit_rate < 85% for 5m | Warning | runbook/cache-hit-rate.md |
| EvictionRateSpike | eviction_rate > 2× baseline for 2m | Warning | runbook/eviction-spike.md |
| LatencyBreach | p99 > NFR threshold for 3m | Critical | runbook/latency-breach.md |
| L2CircuitOpen | circuit_state == 1 for 1m | Critical | runbook/l2-degraded.md |

## Health endpoint
`GET /health`
```json
{
  "status": "UP",
  "liveness": "OK",
  "readiness": {
    "l1_cache": "OK",
    "l2_redis": "OK | DEGRADED | CIRCUIT_OPEN"
  }
}
```

## SLO
- Availability target: 99.9% (error budget: 43.8 min/month)
- SLI: fraction of cache reads returning a response in < p99 threshold
- Burn rate alert: 2× at 1h window (Warning); 5× at 5m window (Critical)

## Dashboard panels
1. Hit rate % (L1 and L2) — time series
2. Eviction rate — time series
3. p50 / p95 / p99 read latency — time series
4. L2 circuit breaker state — stat panel
5. L1 capacity used — gauge

## Observability confidence
| Item | Confidence % | Evidence | HITL |
|------|--------------|----------|------|

**Overall confidence**: NN%
**HITL summary**: N required / N recommended / N optional
**Human review queue**:
- [ ] [validation question per Required item]
```
