# Phase 10 — Node.js Security

## 1. Threat-model first

Security is not a list of magic middleware packages.

Ask:
- what are we protecting?
- who are the attackers?
- what can be stolen?
- what happens if a token is compromised?
- what are the trust boundaries?

## 2. Input validation

All client-controlled input is untrusted.

Validate:
- body
- params
- query
- headers
- files
- webhook payloads

## 3. SQL injection

Never build SQL by string concatenating untrusted input.

Bad:

```js
const query = `SELECT * FROM users WHERE id = ${req.params.id}`;
```

Use parameterized queries/prepared statements.

## 4. NoSQL injection

Do not blindly pass client objects into database operators/queries.

Validate schemas and explicitly construct database filters.

## 5. Command injection

Never directly pass untrusted input into shell commands.

If a system command is genuinely required:
- avoid shell interpolation
- use safe argument APIs
- restrict allowed values
- run with least privilege

## 6. Path traversal

Input such as:

```text
../../secret.txt
```

can escape an intended directory if path handling is unsafe.

Use safe path resolution and allowlists.

## 7. Prototype pollution

JavaScript object manipulation can become dangerous when untrusted keys such as `__proto__` are merged incorrectly.

Use well-maintained libraries and validate/allowlist keys where appropriate.

## 8. CORS

CORS controls whether browsers allow frontend JavaScript from one origin to read responses from another origin.

It is a browser security mechanism.

CORS is NOT:
- authentication
- a server-to-server firewall
- a replacement for authorization

## 9. CSRF

Relevant especially when browser credentials such as cookies are automatically sent.

Use appropriate:
- SameSite settings
- CSRF tokens
- origin validation

## 10. XSS

Prevent attacker-controlled script execution with:
- safe rendering
- output encoding
- CSP
- avoiding unsafe HTML injection
- framework security features

## 11. Security headers

Depending on architecture, security headers can reduce browser-side risks.

Common examples:
- Content-Security-Policy
- Strict-Transport-Security
- X-Content-Type-Options
- Referrer-Policy
- frame-ancestors via CSP

## 12. TLS

HTTPS protects data in transit.

Do not say HTTPS encrypts your database or fixes application vulnerabilities.

## 13. Secrets management

Avoid:

```js
const password = "super-secret";
```

Use:
- environment/configuration management
- secret managers
- least-privilege credentials
- rotation

For cloud systems, services such as AWS Secrets Manager can be appropriate.

## 14. Rate limiting

Protect sensitive endpoints:
- login
- password reset
- expensive APIs
- public APIs

Use distributed/shared state when horizontally scaled.

## 15. Dependency security

Keep dependencies:
- updated
- audited
- minimized
- pinned/controlled appropriately

Do not install packages blindly.

## 16. Logging security

Never casually log:
- passwords
- access tokens
- refresh tokens
- API keys
- secrets
- unnecessary sensitive user data

## 17. Principle of least privilege

Give:
- users only needed permissions
- services only needed cloud/database permissions
- database users only needed operations

## 18. Secure defaults

Examples:
- HTTPS
- restrictive CORS
- safe cookie flags
- body size limits
- timeouts
- validation
- authorization by default for protected routes
