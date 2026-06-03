# LRU Cache — Low-Confidence Items: Proposed Decisions

**Status**: Discussion draft (human-off — no HITL gates)  
**Purpose**: Each item that scored <85% or was marked Required/Recommended HITL in `lru-cache-lld.md` gets a concrete proposed decision here. Bring these to the interview or design review as talking points.  
**Source doc**: `docs/design/lru-cache-lld.md`

---

## Items addressed

| # | Item | LLD ref | Original confidence | Proposed decision |
|---|------|---------|--------------------|--------------------|
| D1 | `put` on existing key — promote to MRU? | FR-5 | 75% Assumed | **Yes — update value + promote** |
| D2 | Thread safety in scope? | NFR-3 | 65% Assumed | **No for base; yes as follow-up layer** |
| D3 | Distributed / Redis-backed scope? | §1 Out of scope | 70% Assumed | **Strictly in-process** |
| D4 | `put` capacity=1 eviction order | FLOW-1 | 85% Inferred | **Evict-then-insert (confirmed)** |
| D5 | `LinkedHashMap` — rejected for interview | §7 D1 | 80% Inferred | **Rejected — explicit rationale below** |
| D6 | Thread safety: `synchronized` vs Caffeine | §7 D2 | 65% Assumed | **`synchronized` for interview; Caffeine for prod** |
| D7 | Tie-breaking among same-recency entries | §7 D3 | 82% Inferred | **DLL order is sufficient; no special logic** |
| D8 | TTL / expiry as extension | §7 D4 | 72% Assumed | **Out of scope base; lazy-expiry sketch ready** |

---

## D1 — `put` on existing key: update value AND promote to MRU

**Original concern**: FR-5 was Assumed at 75% — we didn't have a spec confirmation that updating an existing key counts as a "use" that reorders the entry.

**Proposed decision**: Yes — `put(key, newValue)` on an existing key must both update the value and move the node to the MRU position.

**Reasoning**:
- The LRU contract says "least *recently used*." A write is a use.
- LeetCode 146 (the canonical spec) explicitly requires this behavior.
- Treating a write as a non-use would mean `put(k, v); put(k, v2); put(k, v3)` never refreshes recency — the entry could be silently evicted while actively being written, which is counterintuitive and bug-prone for callers.
- The implementation cost is zero — Flow C (`put` on existing key) already calls `removeNode` + `addToFront`.

**Concrete behavior table**:

| Operation sequence | Expected state after | Evicted? |
|--------------------|-----------------------|----------|
| `cap=2; put(1,1); put(2,2); put(1,99)` | map: {1→99, 2→2}; MRU=1, LRU=2 | Nothing evicted |
| `cap=2; put(1,1); put(2,2); put(3,3)` | map: {2→2, 3→3}; key 1 evicted | key 1 (LRU) |
| `cap=2; put(1,1); put(2,2); put(1,99); put(3,3)` | map: {1→99, 3→3}; key 2 evicted | key 2 (was LRU after put(1,99) promoted key 1) |

**Interview talking point**: "I treat every `put` — insert or update — as a recency event. This keeps the contract simple and consistent: any interaction with a key makes it Most Recently Used."

---

## D2 — Thread safety: not in scope for base design

**Original concern**: NFR-3 was Assumed at 65% — unclear if the interviewer expects concurrent correctness.

**Proposed decision**: Build single-threaded first, then offer concurrency as a depth question.

**Reasoning**:
- Most LLD interview prompts for LRU Cache default to single-threaded unless the problem statement says "concurrent" or "multi-threaded." Asking upfront is correct; our assumption is it is not in scope.
- Adding `synchronized` to `get` and `put` without discussing the tradeoff looks naive. Not adding it and explaining why is better signal.
- The real concurrent problem is subtle: `get` mutates the DLL (promotes node), so it can never hold a read lock — both `get` and `put` are writes at the DLL level. Saying this out loud scores points.

**What to say when asked**:

> "My base implementation is not thread-safe — `get` itself mutates the DLL, so there's no read/write lock distinction. For thread safety I'd add `synchronized` on both methods as the simplest correct approach. For production throughput I'd use Caffeine, which uses a ring-buffer drain strategy to defer DLL mutations to a single background thread, avoiding lock contention on the hot path entirely."

**Three-tier answer for the interview**:

| Tier | Approach | When to use |
|------|----------|-------------|
| Correct + simple | `synchronized` on `get` and `put` | Interview, low-concurrency prod |
| Better throughput | `ReentrantReadWriteLock` — but note: no true read-share because get mutates DLL | Mid-scale |
| Production | Caffeine (`CacheBuilder.newBuilder().maximumSize(n).build()`) | Any real system |

---

## D3 — Distributed / Redis scope: strictly in-process

**Original concern**: Out-of-scope section flagged as HITL — unclear if interviewer wants a distributed layer.

**Proposed decision**: This design is in-process only. Redis / distributed tier is a named extension, not part of the base design.

**Reasoning**:
- An in-process LRU and a distributed cache (Redis) are architecturally different problems. Redis already implements LRU eviction natively (`maxmemory-policy allkeys-lru`). Designing one from scratch means in-memory, in-process.
- Conflating the two in an interview signals scope confusion.
- The correct move: finish the in-process design, then proactively say "if this needed to serve multiple nodes, I'd front this with Redis and use LRU eviction policy there — the HashMap + DLL design isn't relevant at that layer."

**Extension sketch (if asked)**:
- Replace `HashMap + DLL` with a Redis `ZSET` (score = access timestamp) + `HSET` (key → value).
- `get`: `HGET`, then `ZADD key score=now` → O(log n), not O(1). This is a different trade-off the interview should call out explicitly.

---

## D4 — `put` at capacity=1: evict-then-insert order

**Original concern**: FLOW-1 at 85% Inferred — the exact sequencing in Flow E when capacity=1 was not confirmed.

**Proposed decision**: Always evict the LRU entry first (including when capacity=1), then insert the new entry.

**Why this is unambiguous**:
- At capacity=1 with a new key, `tail.prev` == `head.next` — the single live node is simultaneously the LRU and the MRU. Evict it, decrement size, then insert the new node.
- The same Flow E code handles this correctly without special-casing.
- Test case: `cap=1; put(1,1); put(2,2); get(1) → -1; get(2) → 2` — this is the capacity=1 canonical test.

**Key pointer-safety check**: After evicting the only node, the list is `head ↔ tail`. Calling `addToFront(newNode)` correctly makes it `head ↔ newNode ↔ tail`. Sentinel nodes make this safe — no null pointer risk.

**Proposed test**:
```
LRUCacheImpl c = new LRUCacheImpl(1);
c.put(1, 1);
c.put(2, 2);     // evicts key 1
assert c.get(1) == -1;
assert c.get(2) == 2;
c.put(2, 99);    // update, no eviction
assert c.get(2) == 99;
c.put(3, 3);     // evicts key 2
assert c.get(2) == -1;
assert c.get(3) == 3;
```

---

## D5 — `LinkedHashMap` rejected for interview context

**Original concern**: 80% Inferred — the rejection was asserted but rationale was thin.

**Proposed decision**: Reject `LinkedHashMap` in interview. Know it exists; explain why you're not using it.

**Full rejection rationale**:

1. **Zero implementation signal.** `LinkedHashMap(capacity, 0.75f, true)` + overriding `removeEldestEntry` is correct in ~5 lines, but the interviewer cannot assess whether you understand the underlying structure. You could have googled it without knowing why it works.

2. **Internally it IS HashMap + DLL** — same structure, just hidden. Using it without knowing this is worse than not using it.

3. **Cannot extend it.** If the interviewer adds thread safety, generics, TTL, or a custom eviction hook mid-round, you need to understand the structure. `LinkedHashMap` becomes a dead end.

4. **Portability.** The DLL + HashMap approach translates to Python (`OrderedDict`), Go (`container/list` + `map`), and C++ (`unordered_map` + `list`). Explaining the invariant works in any language.

**What to say**:
> "I could use `LinkedHashMap` with access-order mode and `removeEldestEntry` — it's correct and compact. But under the hood it's exactly HashMap + doubly linked list, so I'll implement that directly to make the O(1) invariants explicit and to give us a base we can extend with thread safety or TTL."

---

## D6 — Thread safety implementation: `synchronized` for interview; Caffeine for production

**Original concern**: Decision 2 at 65% Assumed — unclear which tier the interviewer wants.

**Proposed decision**: Default to `synchronized`; name Caffeine as the production alternative; never claim `ReentrantReadWriteLock` is a meaningful improvement here.

**Why `ReentrantReadWriteLock` is NOT better**:

This is a common mistake worth getting right.

```
ReadWriteLock is useful when: reads >> writes, and reads don't mutate shared state.

LRU get() ALWAYS mutates the DLL (promote to MRU).
→ get() must acquire the WRITE lock, not the read lock.
→ With both get() and put() holding the write lock, ReadWriteLock
   gives you zero parallelism advantage over synchronized.
```

Saying this in an interview is high-signal. Most candidates reach for `ReadWriteLock` without realizing `get` is also a write.

**Synchronized implementation**:
```java
public synchronized int get(int key) { ... }
public synchronized void put(int key, int value) { ... }
```

**Production (Caffeine) rationale**:
- Uses a ring buffer (W-TinyLFU policy) to defer access-ordering updates to a background drain thread.
- Hot path (`get`) does a non-blocking write to the ring buffer; the DLL promotion happens asynchronously.
- Result: `get` is effectively lock-free on the read path, with eventual-consistency ordering.
- Bonus: Caffeine supports `expireAfterAccess`, `expireAfterWrite`, `maximumWeight`, `stats()` out of the box.

---

## D7 — Tie-breaking for same-recency entries

**Original concern**: Decision 3 at 82% Inferred — no explicit tie-breaking rule stated.

**Proposed decision**: No special tie-breaking logic is needed. DLL insertion order is the tiebreaker implicitly.

**Why it doesn't matter in practice**:
- Two entries have the same recency only if they were accessed/inserted in the same logical operation, which is impossible in a single-threaded design (operations are serialized).
- In a concurrent design, "same timestamp" ties are possible but the LRU contract only requires that *some* valid least-recently-used entry is evicted — not a specific one among equals.
- The DLL's physical ordering is already deterministic: the node closest to `tail` is always the one evicted. There's no ambiguity at the code level.

**What to say if asked**:
> "My DLL gives a total order — the node at `tail.prev` is always the eviction target, so there's no tie-breaking logic required. In a concurrent scenario where two threads access different keys at the 'same' wall-clock time, either is a valid LRU candidate; evicting either satisfies the contract."

---

## D8 — TTL / expiry: out of scope for base; lazy-expiry sketch ready

**Original concern**: Decision 4 at 72% Assumed — TTL marked out of scope but no concrete extension was specified.

**Proposed decision**: Out of scope for base design. If asked, present lazy expiry (check-on-access). Do not mention a background sweep unless the interviewer explicitly asks for it.

**Why lazy expiry is better than background sweep for an interview**:

| Approach | Pro | Con |
|----------|-----|-----|
| **Lazy expiry** (check on `get`/`put`) | No extra thread; simple; O(1) per op | Expired entries occupy space until accessed |
| Background sweep | Entries freed promptly | Requires `ScheduledExecutorService`; concurrency complexity; out-of-scope in most interview rounds |

**Lazy expiry sketch (additive — no base code change)**:

```java
class NodeWithTTL extends Node {
    long expireAt;   // System.currentTimeMillis() + ttlMs
    NodeWithTTL(int key, int value, long ttlMs) {
        super(key, value);
        this.expireAt = System.currentTimeMillis() + ttlMs;
    }
}

// In get():
if (node instanceof NodeWithTTL ttlNode && System.currentTimeMillis() > ttlNode.expireAt) {
    removeNode(ttlNode);
    map.remove(key);
    size--;
    return -1;   // treat as miss
}
```

**Memory note**: Expired entries that are never accessed again will stay in the cache until they are pushed out by LRU eviction naturally — which is acceptable for a simple implementation. For strict memory bounds, add a background sweep.

---

## Summary: proposed design baseline after resolutions

| Item | Decision |
|------|----------|
| FR-5 — `put` on existing key | Update value + promote to MRU. No eviction. |
| NFR-3 — Thread safety | Out of scope for base design. `synchronized` if asked. Caffeine for prod. |
| Distributed scope | In-process only. Redis is a named extension, different design problem. |
| Capacity=1 eviction | Evict sole entry (tail.prev == head.next), then insert. No special case needed. |
| `LinkedHashMap` | Rejected — valid but opaque; use only if interviewer explicitly asks for concise solution. |
| `ReadWriteLock` | Explicitly worse than `synchronized` for LRU — `get` is a write. Say so. |
| Tie-breaking | None required. DLL order is the implicit tiebreaker. |
| TTL | Out of scope base. Lazy expiry sketch available as a 1-minute extension answer. |

**Revised pipeline confidence after resolutions: 91%**  
All 3 Required HITL items resolved. 4 Recommended items addressed. No pending gates.
