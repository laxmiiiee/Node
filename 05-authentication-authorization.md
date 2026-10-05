# Phase 5 — Authentication & Authorization

## 1. Authentication vs authorization

Authentication:
> Who are you?

Authorization:
> What are you allowed to do?

```text
Authentication → identity
Authorization  → permission
```

Remember:

```text
401 → authentication problem
403 → authorization problem
```

## 2. Session-based authentication

Flow:

```text
Login
 ↓
Server validates credentials
 ↓
Server creates session
 ↓
Client gets session cookie
 ↓
Future request sends cookie
 ↓
Server looks up session
 ↓
User identified
```

The session can live in:
- Redis
- database
- another shared session store

Avoid process-local memory for horizontally scaled production sessions unless architecture deliberately supports it.

## 3. Token-based authentication

Client sends a credential, often:

```http
Authorization: Bearer <access-token>
```

Server:
1. extracts token
2. verifies it
3. validates relevant claims
4. identifies principal
5. authorizes operation

## 4. JWT structure

JWT commonly has:

```text
HEADER.PAYLOAD.SIGNATURE
```

Header:
```json
{"alg":"RS256","typ":"JWT"}
```

Payload:
```json
{"sub":"123","role":"admin","exp":1780000000}
```

Signature protects integrity/authenticity when correctly verified.

## 5. JWT is not encryption

JWT payloads are generally readable.

Never put:
- passwords
- secrets
- highly sensitive information

in a normal signed JWT merely because it is encoded.

## 6. Signing vs encryption

Signing:
> Detect tampering and authenticate the issuer/key holder.

Encryption:
> Keep contents confidential.

A signed JWT is not automatically confidential.

## 7. Access token

Used to access protected APIs.

Usually short-lived.

Example:

```text
Access token
15 minutes
```

Exact lifetime depends on threat model and product requirements.

## 8. Refresh token

Used to obtain a new access token without forcing the user to log in again.

Typical flow:

```text
Access token expires
       ↓
Client sends refresh request
       ↓
Server validates refresh credential
       ↓
New access token
```

Refresh tokens deserve stronger protection because they can maintain a session for longer.

## 9. Refresh token rotation

Instead of repeatedly reusing one refresh token:

```text
A → B → C → D
```

Each successful refresh invalidates/replaces the previous token.

If an already-used token is replayed, the server may detect token theft/reuse and revoke the token family/session.

## 10. Token storage

Possible browser storage:
- localStorage
- sessionStorage
- cookies

There is no universal answer. Security depends on architecture.

### localStorage
Readable by JavaScript, so XSS can potentially expose a stored token.

### HttpOnly cookie
JavaScript cannot directly read it.

Common secure cookie attributes:
- `HttpOnly`
- `Secure`
- appropriate `SameSite`

Cookies automatically participate in browser credential behavior, so CSRF must be considered.

## 11. XSS

Cross-Site Scripting means attacker-controlled JavaScript executes in the application's security origin/context.

Consequences can include:
- reading accessible browser data
- performing actions as the user
- token theft if tokens are JS-accessible

Defenses:
- output encoding
- safe DOM APIs
- CSP
- input handling
- framework protections
- avoiding unsafe HTML injection

## 12. CSRF

Cross-Site Request Forgery abuses automatically attached browser credentials to make unwanted authenticated requests.

Defenses can include:
- SameSite cookies
- CSRF tokens
- Origin/Referer checks where appropriate
- carefully designed CORS
- avoiding unsafe state-changing GET endpoints

## 13. Authentication middleware

Conceptually:

```js
function authenticate(req, res, next) {
    const header = req.headers.authorization;

    if (!header) {
        return res.status(401).json({
            message: "Authentication required"
        });
    }

    const token = header.split(" ")[1];

    try {
        const payload = verifyToken(token);

        req.user = payload;

        next();
    } catch {
        return res.status(401).json({
            message: "Invalid token"
        });
    }
}
```

Important:
- decoding is not verification
- verify signature
- check expiration
- validate expected claims
- do not trust arbitrary client claims

## 14. RBAC

Role-Based Access Control.

Example:

```text
ADMIN
  ├── users.read
  ├── users.create
  ├── users.delete
  └── reports.read

USER
  ├── profile.read
  └── orders.create
```

## 15. Role vs permission

Role:
```text
ADMIN
```

Permission:
```text
users.delete
```

A role can contain many permissions.

Permission-based authorization is often more flexible than hardcoding every role in every route.

## 16. Authorization middleware

```js
function authorize(permission) {
    return (req, res, next) => {
        if (!req.user.permissions.includes(permission)) {
            return res.status(403).json({
                message: "Forbidden"
            });
        }

        next();
    };
}
```

Usage:

```js
app.delete(
    "/users/:id",
    authenticate,
    authorize("users.delete"),
    deleteUser
);
```

## 17. OAuth 2.0

OAuth 2.0 is primarily an authorization framework for delegated access.

It allows a client to obtain access to protected resources without receiving the resource owner's password.

## 18. OAuth vs OIDC

OAuth 2.0:
> authorization

OpenID Connect:
> authentication/identity layer built on OAuth 2.0

This is a frequent interview trap.

## 19. Authorization Code flow

Simplified:

```text
User
 ↓
Application
 ↓
Authorization Server
 ↓
Login + consent
 ↓
Authorization code
 ↓
Application/backend
 ↓
Token endpoint
 ↓
Access token
```

The authorization code is not itself the access token.

## 20. PKCE

PKCE adds a proof to the authorization-code flow.

Conceptually:

```text
code_verifier
     ↓
code_challenge
```

Authorization request carries the challenge.

Token exchange carries the verifier.

The authorization server verifies the relationship.

PKCE is especially important for public clients such as SPAs and mobile apps.

## 21. Okta mental model

Okta can serve as an identity provider/authorization server depending on architecture.

Typical flow:

```text
User
 ↓
Application
 ↓
Okta
 ↓
Authentication
 ↓
Tokens
 ↓
Application/API
```

The important interview skill is explaining:
- who authenticates the user
- who issues tokens
- who validates them
- what claims are trusted
- how APIs authorize access

## 22. Security mistakes

Avoid:
- decoding JWT and trusting it without verification
- putting passwords in JWT
- assuming JWT is encrypted
- logging access/refresh tokens
- long-lived access tokens without reason
- treating authentication as authorization
- returning 401 for authorization failures
- storing sensitive credentials without a threat-model-aware strategy
