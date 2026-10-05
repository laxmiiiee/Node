# Phase 8 — Databases & Node.js

## 1. Connection lifecycle

Do not create a brand-new database connection for every request if the database/client library expects pooling.

Typical architecture:

```text
Node process
   ↓
connection pool
   ├── connection 1
   ├── connection 2
   ├── connection 3
   └── connection N
        ↓
     Database
```

## 2. Connection pooling

Benefits:
- reuse connections
- lower connection setup overhead
- control concurrency
- protect database from unlimited connection creation

Pool size is a tuning parameter, not "bigger is always better."

## 3. Transactions

A transaction groups operations into a unit of work.

Typical properties represented by ACID:

### Atomicity
All-or-nothing.

### Consistency
Transactions preserve defined database invariants/constraints.

### Isolation
Concurrent transactions should behave according to the chosen isolation level.

### Durability
Committed changes survive appropriate failures.

## 4. Example transaction

Suppose transferring money:

```text
Debit A
Credit B
```

You don't want:

```text
Debit succeeds
Credit fails
```

leaving inconsistent state.

Use a transaction where supported.

## 5. Isolation

Concurrency can cause anomalies such as:
- dirty reads
- non-repeatable reads
- phantom reads

Different databases/isolation levels provide different guarantees.

Do not claim all databases behave identically.

## 6. Indexes

Indexes speed up suitable queries by providing a more efficient access path.

But indexes cost:
- storage
- write overhead
- maintenance

Example:

```sql
CREATE INDEX idx_users_email ON users(email);
```

If the application frequently searches by email, an index may be valuable.

## 7. Composite indexes

For queries involving multiple columns, a composite index can be useful.

Example:

```sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status);
```

Index design depends on actual query patterns.

## 8. N+1 problem

Example:
1 query gets 100 users.
Then code performs 1 query per user to get orders.

Total:

```text
1 + 100 = 101 queries
```

This can be expensive.

Solutions:
- joins
- batching
- eager loading
- DataLoader-style batching
- carefully designed queries

## 9. Pagination

Offset pagination:

```sql
SELECT *
FROM users
ORDER BY id
LIMIT 20 OFFSET 1000;
```

Simple, but deep offsets can become expensive and can behave poorly under changing data.

Cursor/keyset pagination:

```text
WHERE id > last_seen_id
ORDER BY id
LIMIT 20
```

Often scales better for large datasets.

## 10. SQL vs NoSQL

SQL databases:
- relational model
- strong schema/constraints
- joins
- transactions
- powerful querying

NoSQL databases vary widely:
- document
- key-value
- wide-column
- graph

Do not say "NoSQL has no schema." Many NoSQL systems still have application-level schemas and validation.

## 11. MongoDB mental model

Document-oriented database.

Typical document:

```json
{
  "_id": "123",
  "name": "Lakshmi",
  "skills": ["Node", "React"]
}
```

Embedding vs referencing is a modeling decision based on:
- access patterns
- cardinality
- document size
- update frequency
- consistency needs

## 12. ORM / ODM

ORM:
> object-relational mapping

ODM:
> object-document mapping

They can improve developer productivity but do not remove the need to understand SQL/database behavior.

## 13. Database errors

Differentiate:
- validation errors
- unique constraint violations
- deadlocks
- connection failures
- timeout
- transient errors

Map them to safe API responses.

## 14. Database performance debugging

If API is slow:
1. measure API latency
2. identify DB portion
3. inspect query plans
4. check indexes
5. check connection pool
6. check locks/contention
7. check N+1
8. inspect payload size
9. inspect network latency
10. measure again after changes
