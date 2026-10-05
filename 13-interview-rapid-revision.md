# Phase 13 — Node.js Rapid Interview Revision

## 1. Core one-liners

### What is Node.js?
A JavaScript runtime built around V8 with an event-driven, asynchronous I/O-oriented architecture.

### Why is Node good for I/O-heavy systems?
Because asynchronous I/O lets the main JavaScript thread continue handling other work while operations are in progress.

### Is Node single-threaded?
JavaScript execution in a Node process uses a main thread, but Node can use OS/runtime mechanisms and worker threads/processes for other work.

### What blocks Node?
Synchronous CPU-heavy JavaScript and blocking operations on the main event-loop thread.

### What is Express?
A web framework on Node that provides routing, middleware and HTTP application abstractions.

### What is middleware?
A function in the request-processing pipeline that can modify the request/response, end the response, call `next()`, or pass an error.

### What is `next()`?
It transfers control to the next middleware/handler in the chain.

### What is `req.params`?
Values captured from route path parameters.

### What is `req.query`?
Query-string parameters.

### What is `req.body`?
Parsed request body, assuming the appropriate body parser/middleware is configured.

### 401 vs 403?
401 authentication failure; 403 authorization failure.

### JWT encrypted?
Normally no. A signed JWT is not the same as an encrypted token.

### Authentication vs authorization?
Authentication verifies identity; authorization checks permissions.

### Access vs refresh token?
Access token accesses APIs; refresh token obtains new access tokens.

### OAuth vs OIDC?
OAuth is authorization; OIDC adds authentication/identity on top of OAuth 2.0.

### Why use refresh tokens?
To allow short-lived access tokens without forcing frequent user login.

### Why rotate refresh tokens?
To reduce replay risk and detect reuse/theft patterns.

### What is RBAC?
Authorization based on roles and their permissions.

### What is a transaction?
A database unit of work providing defined atomicity/consistency/isolation/durability behavior.

### What is connection pooling?
Reusing a bounded pool of database connections rather than creating a new connection for every request.

### What is N+1?
One query retrieves a collection and then one additional query runs per item, causing excessive database round trips.

### What is cache-aside?
Read cache; on miss read DB; store result in cache; return.

### What is backpressure?
A mechanism for preventing a fast producer from overwhelming a slower consumer.

### What is a stream?
An abstraction for incrementally reading/writing data.

### What is a Buffer?
An in-memory representation of binary data.

## 2. Scenario questions

### Scenario: 500 requests suddenly become slow.
Investigate:
1. CPU
2. event-loop lag
3. DB latency
4. external dependencies
5. connection pool
6. memory/GC
7. network
8. traffic pattern

### Scenario: CPU is 95%, DB is normal.
Likely investigate:
- CPU-heavy JavaScript
- expensive serialization
- expensive parsing
- regex
- algorithmic complexity
- synchronous work

Potential solution:
- optimize
- worker threads
- background jobs
- separate service/process
- horizontal scaling

### Scenario: API is authenticated but user gets 403.
Authentication succeeded; authorization failed.

### Scenario: JWT is expired.
Return an authentication failure; client may use a valid refresh mechanism to obtain a new access token.

### Scenario: Database query is slow.
Use measurement and query analysis:
- query plan
- indexes
- locks
- connection pool
- data volume
- N+1
- pagination

### Scenario: 1000 users hit an endpoint after cache expiration.
Potential cache stampede. Consider request coalescing, locks, TTL jitter, stale-while-revalidate, or background refresh.

### Scenario: downstream API is down.
Use:
- timeout
- bounded retries with backoff/jitter when appropriate
- circuit breaker/fallback where appropriate
- clear error mapping
- observability

## 3. Your interview architecture answer

For:

> "How would you design a Node API?"

A strong structure:

```text
Client
 ↓
Load balancer/API gateway
 ↓
Node/Express
 ↓
logging/tracing
 ↓
authentication
 ↓
authorization
 ↓
validation
 ↓
controller
 ↓
service
 ↓
repository
 ↓
DB/cache
```

For slow/background work:

```text
API
 ↓
Queue
 ↓
Worker
 ↓
DB/object storage
```

## 4. What to say about your real experience

Your strongest discussion points are:
- React + Node full-stack development
- JWT access/refresh tokens
- OAuth integration
- RBAC
- Okta authentication
- API security
- rate limiting
- HTTPS/TLS
- AWS Secrets Manager
- transactional expense APIs
- AWS event-driven components where applicable

Don't merely list technologies.

Explain:
1. the problem
2. architecture
3. why you chose the mechanism
4. request flow
5. security considerations
6. failure cases
7. trade-offs
8. testing

## 5. Interview answer formula

For technical questions:

```text
Definition
 ↓
How it works
 ↓
Example
 ↓
Trade-off
 ↓
Production concern
```

Example:

> What is JWT?

1. Definition: signed token format carrying claims.
2. How: header + payload + signature.
3. Example: bearer access token.
4. Trade-off: stateless verification vs revocation complexity.
5. Production: short expiry, secure handling, verification, refresh strategy.

## 6. Final two-day priority

Highest priority:
1. Event loop
2. async/await + Promise behavior
3. HTTP
4. Express middleware
5. authentication/authorization
6. JWT
7. OAuth/OIDC
8. error handling
9. database fundamentals
10. caching/Redis
11. security
12. API/system design
13. your own project architecture

Then move to:
- JavaScript
- TypeScript
- DSA

The goal is confident explanation, not memorizing every API method.
