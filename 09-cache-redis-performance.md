# Phase 9 — Caching, Redis & Performance

## 1. Why cache?

Caching avoids repeatedly performing expensive work.

Example:

```text
Request
 ↓
Cache lookup
 ├── hit → return cached value
 └── miss → database → cache result
```

## 2. Cache-aside

Common pattern:

```text
Read
 ↓
Check cache
 ↓
miss
 ↓
DB
 ↓
store result in cache
 ↓
return
```

On writes, cache invalidation/update must be designed carefully.

## 3. Cache invalidation

Classic problem:

> Cached data can become stale.

Strategies:
- TTL
- invalidate on writes
- versioned keys
- write-through
- explicit refresh

## 4. Redis

Redis is an in-memory data store often used for:
- caching
- sessions
- rate limiting
- distributed locks in carefully designed cases
- queues/streams depending on architecture
- counters

## 5. TTL

Time-to-live automatically expires a cache entry.

Useful for:
- short-lived data
- sessions
- rate-limit windows
- avoiding indefinite staleness

## 6. Rate limiting with Redis

A distributed application may have:

```text
Server A ─┐
Server B ─┼→ Redis counter
Server C ─┘
```

This provides shared state for rate-limit algorithms.

## 7. Performance debugging

Never optimize by guessing.

Measure:
- request latency
- throughput
- CPU
- memory
- event-loop lag
- database latency
- external API latency
- cache hit rate
- garbage collection behavior
- network latency

## 8. Event-loop lag

If the event loop is blocked:
- callbacks are delayed
- timers are delayed
- HTTP responses are delayed

Monitor event-loop delay in production.

## 9. Horizontal scaling

If one Node process cannot handle the workload:

```text
Load Balancer
 ├── Node A
 ├── Node B
 ├── Node C
 └── Node D
```

Stateless API design makes this easier.

Shared state should live in appropriate external systems when required.

## 10. Cache stampede

If a popular cache entry expires and thousands of requests simultaneously hit the DB:

```text
Cache expires
   ↓
1000 requests
   ↓
1000 DB queries
```

Mitigations can include:
- locking
- request coalescing
- jittered expiration
- background refresh
- stale-while-revalidate patterns

## 11. Memory leaks

Symptoms:
- memory steadily grows
- GC becomes frequent
- latency increases
- process eventually crashes

Common causes:
- global collections that grow forever
- event listeners never removed
- timers retaining objects
- closures retaining large data
- caches without eviction

## 12. Backpressure and performance

If producers outrun consumers, uncontrolled buffering increases memory usage.

Use streams/queues/backpressure-aware designs where appropriate.
