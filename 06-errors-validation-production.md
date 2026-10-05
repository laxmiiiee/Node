# Phase 6 — Errors, Validation & Production API Practices

## 1. Error categories

### Operational errors
Expected runtime failures:
- invalid input
- database unavailable
- timeout
- upstream service failure
- not found
- conflict

These can often be handled gracefully.

### Programming errors
Unexpected bugs:
- TypeError
- broken assumptions
- invalid state
- logic bugs

These require investigation and may indicate the process is unsafe to continue.

## 2. Central error handling

A common architecture:

```text
Controller
   ↓
throw/pass error
   ↓
central error middleware
   ↓
map to safe HTTP response
```

Example:

```js
app.use((err, req, res, next) => {
    console.error(err);

    res.status(500).json({
        message: "Internal Server Error"
    });
});
```

## 3. Error response design

A consistent error format is easier for frontend clients.

Example:

```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User not found"
  }
}
```

Avoid leaking internal implementation details.

## 4. Validation

Validate all external input:
- body
- path parameters
- query parameters
- headers where relevant
- uploaded files
- webhook payloads

Never trust the client.

## 5. Authorization after authentication

Validation and authentication are different.

```text
Request
 ↓
Parse
 ↓
Authenticate
 ↓
Authorize
 ↓
Validate business input
 ↓
Business logic
```

Exact ordering can vary by application, but do not confuse the responsibilities.

## 6. Rate limiting

Rate limiting controls how many requests a client can make over a period.

Useful for:
- brute-force login attempts
- abuse
- expensive endpoints
- public APIs

In a distributed deployment, use a shared mechanism such as Redis rather than relying only on per-process memory.

## 7. Timeouts

Never assume upstream services will always respond.

Use:
- client timeouts
- database timeouts
- queue visibility/deadline controls
- cancellation/abort mechanisms where supported

A request that waits forever can consume resources.

## 8. Retries

Retries can help transient failures.

But retrying blindly is dangerous.

Consider:
- whether the operation is idempotent
- exponential backoff
- jitter
- maximum attempts
- server load
- error classification

Bad retry behavior can create retry storms.

## 9. Graceful shutdown

A production Node service should respond properly to termination signals.

Conceptually:

```text
SIGTERM
 ↓
stop accepting new work
 ↓
finish/drain existing requests where possible
 ↓
close database connections
 ↓
close consumers/listeners
 ↓
exit
```

This is especially important during deployments.

## 10. Configuration

Use environment/configuration management for:
- ports
- database URLs
- service endpoints
- feature flags
- credentials

Do not commit secrets.

For sensitive production credentials, use a secret manager where appropriate.

## 11. Health checks

Typical:
- liveness: is the process alive?
- readiness: can it safely receive traffic?

Do not make liveness unnecessarily dependent on every downstream dependency.

A readiness check may reasonably include important dependency health depending on architecture.

## 12. Logging

Good logs answer:
- what happened?
- when?
- where?
- request/correlation ID?
- user/request context where appropriate?
- error class?
- duration?

Avoid:
- passwords
- tokens
- secrets
- unnecessary PII

## 13. Correlation/request IDs

A request ID lets you trace:

```text
Frontend
 ↓
API gateway
 ↓
Node service
 ↓
database
 ↓
another service
```

through logs.

## 14. Production mindset

For every API, think about:

```text
Validation
Authentication
Authorization
Timeouts
Rate limits
Error handling
Logging
Metrics
Tracing
Retries
Idempotency
Security
Graceful shutdown
```
