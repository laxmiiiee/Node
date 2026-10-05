# Phase 12 — Node.js System Design & Architecture

## 1. Basic scalable architecture

```text
Client
  ↓
CDN / Load Balancer / API Gateway
  ↓
Node API instances
  ↓
Service layer
  ↓
Database
  ↓
Cache / Redis
  ↓
Queue / workers
```

The exact components depend on requirements.

## 2. Stateless Node APIs

Prefer keeping request-specific state outside the individual Node process when horizontal scaling is needed.

```text
Load Balancer
 ├── Node A
 ├── Node B
 └── Node C
```

Shared state:
- database
- Redis
- object storage
- queues

## 3. Background jobs

Don't make users wait for slow work when it doesn't need to happen synchronously.

Example:

```text
POST /reports
 ↓
validate
 ↓
create job
 ↓
202 Accepted
 ↓
queue
 ↓
worker
 ↓
generate report
 ↓
store result
```

Useful for:
- emails
- reports
- image processing
- large exports
- notifications
- expensive computations

## 4. Queues

Benefits:
- decouple producers/consumers
- smooth traffic spikes
- retry failed work
- move slow work off request path

Important concepts:
- visibility timeout
- retry policy
- dead-letter queue
- idempotency
- ordering requirements
- duplicate delivery

## 5. At-least-once delivery

Many queue systems can deliver a message more than once.

Therefore consumers should often be idempotent.

Example:

```text
payment event
 ↓
process payment
 ↓
record event ID
```

If duplicate arrives:

```text
event ID already processed
 ↓
do not repeat side effect
```

## 6. API idempotency keys

For operations such as payments/order creation, clients can send:

```http
Idempotency-Key: abc123
```

Server stores the result associated with that key.

If the same request is retried:

```text
same key
 ↓
return previous result
```

This prevents duplicate side effects.

## 7. Timeouts and cancellation

Every external dependency should have an appropriate timeout.

For fetch-like APIs, use cancellation mechanisms such as `AbortController` where supported.

Timeouts prevent one slow dependency from tying up resources indefinitely.

## 8. Retries

Use:
- exponential backoff
- jitter
- bounded attempts
- error classification

Do not retry permanent failures.

## 9. Circuit breaker concept

If a downstream service is failing repeatedly:

```text
Normal
 ↓
Failures increase
 ↓
Circuit opens
 ↓
Fast fail
 ↓
After recovery window
 ↓
Half-open test
 ↓
Close if healthy
```

This can protect your service from wasting resources on a broken dependency.

## 10. Database bottlenecks

If API latency is high:
- check query latency
- query plan
- indexes
- connection pool
- locks
- N+1
- payload size
- database CPU/memory

## 11. Caching

Use caching for data that:
- is expensive to compute/fetch
- is read frequently
- can tolerate appropriate staleness

Never introduce cache without a consistency/invalidation plan.

## 12. Horizontal scaling

Node is easy to scale horizontally when:
- APIs are stateless
- sessions/shared state are externalized
- shared caches are used appropriately
- database is designed for the workload

## 13. Failure thinking

For every component ask:

> What happens if it is slow?
> What happens if it is unavailable?
> What happens if it returns malformed data?
> What happens if the request is duplicated?
> What happens if the process crashes?

That mindset separates basic API coding from system design.
