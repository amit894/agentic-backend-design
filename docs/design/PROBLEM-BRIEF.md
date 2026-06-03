# Problem Brief

Fill this in before running `/lld-round`. The LLD workflow reads this as the source of truth for requirements, API design, and trade-offs.

---

# Problem Brief

Fill this in before running `/lld-round`. The LLD workflow reads this as the source of truth for requirements, API design, and trade-offs.


## Title

LRU (Least Recently Used) Cache — Production Configuration

## One-line summary

A high-throughput, low-latency in-memory LRU cache with multi-tier support, TTL policies, concurrency sharding, observability hooks, and resilience primitives — ready for a senior principal engineer to tune and ship.

## Background

At scale, services repeatedly pay the cost of fetching the same data from slow backing stores (relational DBs, remote APIs, object storage). An LRU cache absorbs this read pressure by keeping hot entries in memory and evicting the least recently used entry when capacity is exceeded. For high-throughput services (>50k RPS), a naive single-lock cache becomes a bottleneck; the design must account for shard-level locking, async write paths, and multi-tier fallback (L1 heap → L2 Redis/Memcached) to sustain sub-millisecond p99 reads.

## Users & actors

| Actor                          | Goal                                                                         |
| ------------------------------ | ---------------------------------------------------------------------------- |
| Application Service            | Read cached data with sub-millisecond p99 latency                            |
| Application Service            | Write (populate) cache entries on cache-miss or pre-warm                     |
| Cache Eviction Worker          | Evict LRU entries in configurable batch sizes when capacity is exceeded       |
| L2 Cache (Redis / Memcached)   | Act as overflow tier when L1 heap reaches capacity                           |
| Backend DB / External Service  | Source of truth on full cache miss (both L1 and L2 miss)                     |
| Monitoring / Alerting System   | Consume hit-rate, eviction-rate, and latency histograms via Prometheus / OTel|
| Platform / SRE Engineer        | Tune shard count, TTLs, and circuit-breaker thresholds per environment       |

## Core scenarios (happy path)

1. **Cache hit (L1)** — Application calls `get(key)`; entry exists in the in-process shard, LRU order is updated, value is returned in <1ms with zero network I/O.
2. **Cache miss → L2 promoted to L1** — Entry is absent from L1 but found in the Redis L2 tier; it is deserialized, inserted into L1 (evicting LRU entry if full), and returned to the caller transparently.
3. **Cache miss → DB load → dual write** — Entry absent from both tiers; application fetches from the backing store, writes to L2 (async), writes to L1, and returns the value; subsequent requests for the same key hit L1.
4. **Capacity-triggered eviction** — L1 reaches `max_entries`; the eviction worker removes `eviction_batch_size` LRU entries in a single pass, emitting an eviction histogram metric and an audit log line per entry if `log_evictions: true`.
5. **Negative cache hit** — A previous lookup returned "not found"; the negative-cache entry (TTL = 5s) is returned immediately, preventing a thundering-herd stampede to the backing store.
6. **Circuit breaker open on L2 failure** — Redis becomes unavailable; after `failure_threshold` consecutive errors the circuit opens, all L2 calls are skipped, and L1 continues serving reads in degraded mode until `reset_timeout_seconds` elapses.

## Constraints

- **Scale**: 100k RPS peak per node; L1 capacity = 500k entries (~2 GB heap); L2 capacity = 50M entries (Redis cluster, 64 GB).
- **Latency**: p99 < 1ms for L1 reads; p99 < 5ms for L2 reads; eviction batch must complete in <10ms to avoid GC pressure spikes.
- **Consistency**: Eventual — stale reads permitted within TTL window; no cross-node cache coherence required (each node runs an independent L1).
- **Concurrency**: 256 shards (power-of-2 for bitwise mod); `read_concurrency: true`; async write buffer depth = 1024 to decouple hot-path writes from lock contention.
- **Serialization**: MessagePack (compact binary) with Zstandard compression for entries > 512 bytes.
- **Observability**: Prometheus metrics with OTel trace context propagation; p50/p95/p99 histograms for hit latency and eviction latency.
- **Persistence**: Disabled in hot path; optional async snapshot flush every 60s for warm-restart acceleration.
- **Budget / infra**: Cloud-native (AWS/GCP); L2 is a managed Redis cluster; L1 runs in-process on each application pod.
- **Team / timeline**: Platform/infra team; this config is the canonical baseline — app teams fork and override per-service via environment overlays.

## Known integrations

- **Redis 7.x** (L2 cache tier, TLS-enabled, connection pool)
- **Prometheus + Grafana** (metrics scrape endpoint on `:9090/metrics`)
- **OpenTelemetry Collector** (trace context propagation for cache-miss spans)
- **Kubernetes ConfigMap / Helm values** (environment-specific overrides injected at deploy time)
- **Snapshot storage: local PVC or S3** (warm-up snapshot persistence)

## Explicit non-goals (out of scope for MVP)

- No distributed cache invalidation / pub-sub invalidation bus (cache-aside pattern only).
- No write-through or write-behind to the backing DB — the cache layer is read-side only.
- No cache stampede prevention via locking/promise coalescing (handled at the application layer).
- No multi-region replication of cache state.
- No per-key custom TTL API (all TTLs governed by the config; per-key override is a v2 feature).
- No encryption-at-rest for L1 heap entries.

## Open questions for the design round

- Should `sliding_window: true` be the default? It resets TTL on every access, which can cause memory spikes for entries that are accessed continuously but should expire — needs a per-use-case decision.
- What is the acceptable stale-read window for each consumer service? This determines whether `default_seconds: 300` is safe or if shorter TTLs (e.g. 30s) are required for latency-sensitive flows.
- Should `eviction_batch_size` be adaptive (back-pressure driven) or fixed? Fixed is simpler but can cause latency tail spikes under heavy write load.
- Is a 5-second negative-cache TTL safe for all key namespaces, or do some domains (e.g. auth tokens, feature flags) need a shorter or zero negative TTL?
- What monitoring SLA triggers an on-call page? Define hit-rate floor (e.g. < 85%) and eviction-rate ceiling as alerting thresholds before going to prod.

## Interview context

- **Company / round type**: Senior / Principal backend LLD or system design round (e.g. FAANG / FAANG-adjacent, 60 min).
- **Depth expected**: Full API design (get/put/delete/invalidate), internal data structure rationale (doubly-linked list + hash map), concurrency model (shard striping vs. RWMutex), scale discussion (single-node limits → multi-tier → distributed), and code sketch for the core eviction logic.
- **Key trade-offs to articulate**:
    - LRU vs. LFU vs. ARC — why LRU is the default and when to switch.
    - Fixed shard count (simple, predictable) vs. consistent hashing (dynamic, more complex).
    - Synchronous vs. asynchronous L2 write — latency vs. consistency.
    - Sliding window TTL — simplicity vs. memory unboundedness risk.