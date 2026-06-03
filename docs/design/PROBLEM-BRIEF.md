# Problem Brief

Fill this in before running `/lld-round` or `/design-and-ship`. The LLD workflow reads this as the source of truth for requirements, API design, and trade-offs.

---

## Title

LRU (Least Recently Used) Cache — Production Configuration

## Output folder

`docs/design/problems/lru-cache-production`

## One-line summary

A high-throughput, low-latency in-memory LRU cache with multi-tier support (L1 heap + L2 Redis), TTL policies, 256-shard concurrency, Prometheus/OTel observability, and circuit breaker resilience — production-baseline for platform teams at 100k RPS per node.

## Background

At scale, services repeatedly pay the cost of fetching the same data from slow backing stores (relational DBs, remote APIs, object storage). An LRU cache absorbs this read pressure by keeping hot entries in memory and evicting the least recently used entry when capacity is exceeded. For high-throughput services (>50k RPS), a naive single-lock cache becomes a bottleneck — the design must account for shard-level locking, async write paths, and multi-tier fallback (L1 heap → L2 Redis) to sustain sub-millisecond p99 reads.

## Users & actors

| Actor | Goal |
|-------|------|
| Application Service | Read cached data with p99 < 1ms latency |
| Application Service | Write (populate) cache entries on miss or pre-warm |
| Cache Eviction Worker | Evict LRU entries in configurable batch sizes when L1 is at capacity |
| L2 Cache (Redis 7.x) | Act as overflow tier when L1 heap reaches max_entries |
| Backend DB / External Service | Source of truth on full cache miss (both L1 and L2 miss) |
| Monitoring / Alerting System | Consume hit-rate, eviction-rate, and latency histograms via Prometheus / OTel |
| Platform / SRE Engineer | Tune shard count, TTLs, and circuit-breaker thresholds per environment |

## Core scenarios (happy path)

1. **L1 hit** — `get(key)` finds entry in the in-process shard; LRU order updated; value returned in < 1ms with zero network I/O.
2. **L2 promote to L1** — L1 miss; Redis GET succeeds; entry deserialized, inserted into L1 (evicting LRU if full), returned to caller.
3. **Full miss → DB load → dual write** — Both tiers miss; application fetches from backing store; async L2 SET; L1 insert; value returned.
4. **Capacity-triggered eviction** — L1 at `max_entries`; eviction worker removes `eviction_batch_size` LRU entries in one pass; emits Prometheus histogram; audit log line per entry if `log_evictions: true`.
5. **Negative cache hit** — A prior lookup returned not-found; negative entry (TTL = 5s) returned immediately; prevents thundering-herd stampede to backing store.
6. **Circuit breaker on L2 failure** — Redis errors × `failure_threshold`; circuit opens; L1-only degraded mode; circuit resets after `reset_timeout_seconds`.

## Constraints

- **Scale**: 100k RPS peak per node; L1 capacity = 500k entries (~2 GB heap); L2 = Redis cluster 50M entries (64 GB).
- **Latency**: p99 < 1ms for L1 reads; p99 < 5ms for L2 reads; eviction batch < 10ms to avoid GC pressure spikes.
- **Consistency**: Eventual — stale reads permitted within TTL window; no cross-node cache coherence required.
- **Concurrency**: 256 shards (power-of-2 for bitwise mod); `read_concurrency: true`; async write buffer depth = 1024.
- **Serialization**: MessagePack + Zstandard compression for entries > 512 bytes.
- **Observability**: Prometheus metrics; OTel trace context propagation; p50/p95/p99 histograms for hit and eviction latency.
- **Persistence**: Disabled in hot path; optional async snapshot flush every 60s to S3 or local PVC for warm-restart.
- **Budget / infra**: Cloud-native (AWS/GCP); L2 is a managed Redis cluster; L1 runs in-process per application pod.
- **Team / timeline**: Platform/infra team; this is the canonical baseline — app teams override per-service via K8s ConfigMap / Helm values.

## Known integrations

- **Redis 7.x** — L2 cache tier, TLS-enabled, connection pool
- **Prometheus + Grafana** — metrics scrape on `:9090/metrics`
- **OpenTelemetry Collector** — trace context propagation for cache-miss spans
- **Kubernetes ConfigMap / Helm values** — environment-specific overrides at deploy time
- **S3 / local PVC** — warm-restart snapshot persistence

## Explicit non-goals (out of scope for MVP)

- No distributed cache invalidation or pub-sub invalidation bus (cache-aside pattern only).
- No write-through or write-behind to the backing DB — read-side cache only.
- No cache stampede prevention via promise coalescing (handled at the application layer).
- No multi-region replication of cache state.
- No per-key custom TTL API — all TTLs governed by config; per-key override is a v2 feature.
- No encryption-at-rest for L1 heap entries.

## Open questions for the design round

- Should `sliding_window: true` be the default? It resets TTL on every access, which risks memory spikes for continuously-accessed entries.
- What stale-read window is acceptable per consumer service? Determines whether `default_ttl_seconds: 300` is safe or shorter TTLs are needed.
- Should `eviction_batch_size` be fixed or adaptive (back-pressure driven)? Fixed is simpler but can spike eviction latency under heavy write load.
- Is a 5s negative-cache TTL safe for all key namespaces, or do auth tokens and feature flags need zero negative TTL?
- What hit-rate floor and eviction-rate ceiling trigger an on-call page?

## Interview context

- **Round type**: Senior / Principal backend LLD, 60 min, FAANG-style.
- **Depth expected**: API (get/put/delete/invalidate/stats), data structure per shard (DLL + HashMap), concurrency model (shard striping vs global RWMutex), scale discussion (single-node → multi-tier → distributed), code sketch for eviction and circuit breaker logic.
- **Key trade-offs to articulate**:
  - LRU vs LFU vs ARC — why LRU is the default and when to switch.
  - Fixed shard count vs consistent hashing.
  - Synchronous vs asynchronous L2 write — latency vs consistency.
  - Sliding window TTL — simplicity vs memory unboundedness risk.
  - Hand-rolled vs Caffeine (JVM) — interview signal vs production pragmatism.
