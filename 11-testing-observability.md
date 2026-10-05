# Phase 11 — Testing & Observability

## 1. Testing layers

### Unit tests
Test isolated logic.

### Integration tests
Test multiple components together, often including database or external boundaries.

### API/HTTP tests
Test request → response behavior.

### End-to-end tests
Test realistic user/business flows across multiple systems.

## 2. What to test in Node APIs

Test:
- success paths
- validation failures
- authentication failures
- authorization failures
- not found
- conflicts
- downstream failures
- timeouts
- malformed input
- pagination
- concurrency-sensitive behavior where important

## 3. Mocking

Mock dependencies when isolation is useful.

But over-mocking can produce tests that pass while the real system is broken.

Use integration tests for important boundaries.

## 4. Testing async code

Always await asynchronous operations in tests.

Avoid tests that finish before the async assertion executes.

## 5. Observability

Three pillars:

```text
Logs
Metrics
Traces
```

### Logs
Detailed events.

### Metrics
Numerical time-series:
- request count
- error rate
- latency
- CPU
- memory

### Traces
Follow a request across services.

## 6. Golden API metrics

Track:
- throughput
- latency
- errors
- saturation

Also useful:
- p50 latency
- p95
- p99
- cache hit rate
- event-loop lag
- DB latency

## 7. Correlation IDs

A request ID allows you to connect logs across components.

Example:

```text
requestId=abc123
```

appears in:
- API gateway
- Node service
- downstream service
- relevant logs

## 8. Alerting

Alert on meaningful symptoms:
- high error rate
- high latency
- saturation
- unavailable dependencies
- unusual traffic

Avoid alerts that fire constantly and become ignored.
