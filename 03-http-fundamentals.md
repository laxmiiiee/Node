# Phase 3 — HTTP Fundamentals

## 1. HTTP request

A request has:
- method
- URL/path
- headers
- optional body

Example:

```http
POST /api/users HTTP/1.1
Content-Type: application/json
Authorization: Bearer TOKEN

{"name":"Lakshmi"}
```

## 2. HTTP response

A response has:
- status code
- headers
- optional body

```http
HTTP/1.1 201 Created
Content-Type: application/json

{"id":42,"name":"Lakshmi"}
```

## 3. HTTP methods

### GET
Retrieve a resource.

### POST
Create a resource or trigger an operation.

### PUT
Typically replace/set the representation of a resource. Usually idempotent.

### PATCH
Partially modify a resource.

### DELETE
Remove a resource.

## 4. Path params vs query params

```text
/users/42
       ↑
path parameter

/users?page=2&limit=20
       ↑
query parameters
```

Path parameter:
> Which resource?

Query parameter:
> How should I retrieve/filter/shape the result?

## 5. Headers

Examples:
- `Content-Type`
- `Accept`
- `Authorization`
- `User-Agent`
- caching headers
- tracing headers

Headers carry metadata about the HTTP communication.

## 6. Content-Type

Common:
- `application/json`
- `text/plain`
- `multipart/form-data`
- `application/x-www-form-urlencoded`

`multipart/form-data` is common for file uploads.

## 7. Status codes

### 2xx
- `200 OK`
- `201 Created`
- `204 No Content`

### 4xx
- `400 Bad Request`
- `401 Unauthorized`
- `403 Forbidden`
- `404 Not Found`
- `409 Conflict`
- `422 Unprocessable Content`
- `429 Too Many Requests`

### 5xx
- `500 Internal Server Error`
- `502 Bad Gateway`
- `503 Service Unavailable`
- `504 Gateway Timeout`

## 8. 401 vs 403

```text
401 → authentication failure
403 → authorization failure
```

Memory trick:

```text
401: "Who are you?"
403: "I know who you are, but you can't do this."
```

## 9. Node's `http` module

```js
const http = require("node:http");

const server = http.createServer((req, res) => {
    res.statusCode = 200;
    res.setHeader("Content-Type", "text/plain");
    res.end("Hello");
});

server.listen(3000);
```

`req` represents the incoming request.

`res` represents the outgoing response.

## 10. JSON response without Express

```js
res.setHeader("Content-Type", "application/json");

res.end(JSON.stringify({
    message: "success"
}));
```

## 11. Request body is a stream

The body arrives over time.

Conceptually:

```text
HTTP body
  ↓
chunks
  ↓
Buffer
  ↓
string
  ↓
JSON.parse()
  ↓
object
```

## 12. Statelessness

HTTP itself does not remember previous requests.

Applications add state through:
- cookies
- sessions
- access tokens
- refresh tokens
- databases/cache

## 13. REST

REST is an architectural style, not merely "CRUD with GET/POST."

Common characteristics:
- client-server separation
- stateless interactions
- uniform interface
- resource-oriented design
- cacheability
- layered architecture

## 14. Idempotency

An operation is idempotent if repeating it has the same intended effect on the resource state as doing it once.

Typical:
- GET: idempotent
- PUT: idempotent
- DELETE: idempotent by HTTP semantics
- POST: generally not guaranteed idempotent
- PATCH: depends on operation

Idempotent does not mean "response must be identical."

## 15. POST vs PUT

POST often:
```http
POST /users
```

Server determines the new resource ID.

PUT:
```http
PUT /users/123
```

Client identifies the target resource and supplies the desired representation.

## 16. HTTP vs HTTPS

HTTPS is HTTP over TLS.

TLS provides:
- confidentiality
- integrity
- server authentication

It protects data in transit but does not make the application itself secure.
