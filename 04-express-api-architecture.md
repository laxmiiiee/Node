# Phase 4 — Express & API Architecture

## 1. What Express is

Express is a web framework built around Node's HTTP capabilities.

It provides:
- routing
- middleware
- request helpers
- response helpers
- error handling patterns

It does not replace Node.

## 2. Basic application

```js
const express = require("express");

const app = express();

app.listen(3000);
```

## 3. Routes

```js
app.get("/users", getUsers);
app.post("/users", createUser);
app.patch("/users/:id", updateUser);
app.delete("/users/:id", deleteUser);
```

## 4. JSON body parsing

```js
app.use(express.json());
```

Then:

```js
app.post("/users", (req, res) => {
    console.log(req.body);
});
```

Conceptually `express.json()` parses incoming JSON request bodies.

## 5. Middleware

Typical signature:

```js
(req, res, next) => {
    // work
    next();
}
```

Middleware can:
1. end the response
2. call `next()`
3. pass an error

## 6. Why middleware ordering matters

```js
app.use(authenticate);
app.get("/profile", getProfile);
```

Authentication runs before the route.

Ordering is part of Express application behavior.

## 7. `return res...`

Prefer:

```js
if (!authorized) {
    return res.status(403).json({
        message: "Forbidden"
    });
}
```

This prevents execution from accidentally continuing after a response is sent.

## 8. Routers

```js
const router = express.Router();

router.get("/", getUsers);
router.get("/:id", getUser);

app.use("/users", router);
```

This gives:
- `GET /users`
- `GET /users/:id`

## 9. Route params

```js
req.params.id
```

For:

```text
/users/42
```

the value is normally `"42"` as a string.

## 10. Query params

```js
req.query.page
```

For:

```text
/users?page=2
```

values generally arrive as strings or parsed query structures depending on parser/configuration.

Validate/coerce them.

## 11. Request body

```js
req.body
```

Never assume it has the expected shape.

Validate it.

## 12. Controller / service / repository

### Controller
HTTP concerns:
- read request
- call service
- choose response/status
- pass errors

### Service
Business logic:
- business rules
- orchestration
- domain operations

### Repository
Persistence concerns:
- database queries
- persistence abstraction

Flow:

```text
Route
  ↓
Controller
  ↓
Service
  ↓
Repository
  ↓
Database
```

## 13. Why separate layers?

Benefits:
- testability
- maintainability
- reuse
- clearer responsibilities
- easier refactoring

Don't blindly add layers to tiny applications; use architecture appropriate to complexity.

## 14. Async controller

```js
async function createUser(req, res, next) {
    try {
        const user = await userService.createUser(req.body);

        return res.status(201).json(user);
    } catch (err) {
        next(err);
    }
}
```

## 15. Centralized error middleware

Express error middleware has four parameters:

```js
(err, req, res, next)
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

## 16. Never leak internals

Avoid returning:
- stack traces
- SQL queries
- filesystem paths
- secrets
- internal infrastructure details

to clients in production.

Log safely on the server.

## 17. Validation

Validate:
- required fields
- types
- ranges
- formats
- allowed values
- object structure
- query/path parameters

Libraries commonly used in Node ecosystems:
- Zod
- Joi
- Ajv

Validation should happen before business logic relies on input.

## 18. Validation vs sanitization

Validation:
> Is the input acceptable?

Sanitization/normalization:
> How should we transform or normalize it for a particular use?

Context matters. Do not blindly "strip dangerous characters" because that can corrupt legitimate data.

## 19. Complete Express flow

```text
HTTP request
    ↓
Global middleware
    ↓
body parsing
    ↓
logging/tracing
    ↓
authentication
    ↓
route matching
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
database
    ↓
response

Errors
    ↓
central error middleware
```

## 20. Interview question

Q: Why use Express if Node already has `http`?

Answer:

> Node's `http` module gives low-level HTTP primitives. Express provides higher-level abstractions such as routing, middleware composition, request/response helpers, and a common application structure, which makes building and maintaining HTTP services easier.
