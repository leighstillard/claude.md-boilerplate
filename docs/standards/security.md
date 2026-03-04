# Security Standards

## DO

- Parameterise all database queries. Use ORM query builders or prepared statements.
- Validate and sanitise all input at API boundaries. Prefer allowlists (known good values) over denylists.
- Encode all output to prevent XSS. Use framework-provided templating that auto-escapes.
- Implement auth as centralised middleware or interceptor. One place to enforce, one place to audit.
- Use CSRF tokens on all state-changing endpoints.
- Set security headers on all responses: `Strict-Transport-Security`, `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`.
- Apply rate limiting at the application layer, even if also applied at infrastructure. Use token bucket or sliding window.
- Validate file uploads: check MIME type, enforce size limits, sanitise filenames. Store outside webroot.
- Use constant-time comparison for security-sensitive string checks (tokens, hashes).
- Pin dependencies to exact versions. Use lockfiles. Review diffs on updates.

## DO NOT

- Do not trust client-side validation as a security control. Always re-validate server-side.
- Do not roll custom cryptography. Use well-known libraries (Go: `crypto/`, Python: `cryptography`, Node: `crypto`).
- Do not use MD5 or SHA1 for anything security-sensitive. Use SHA-256+ for hashing, bcrypt/scrypt/argon2 for passwords.
- Do not log full request/response bodies without scrubbing sensitive fields first.
- Do not deserialise untrusted input without schema validation (no raw `pickle`, no unvalidated JSON-to-struct).
- Do not expose stack traces, internal paths, or dependency versions in error responses.
- Do not disable TLS verification, even in dev. Use proper certs or mkcert for local development.
- Do not store sensitive data in URL query parameters (they appear in logs, referrer headers, browser history).

## REQUIRE (external dependency)

- If no authentication provider is configured, flag to user: "Auth middleware has no backing identity provider. This must be configured before any protected endpoint is functional."
- If no TLS certificate is available for the environment, flag to user: "TLS must be configured at the infrastructure layer. Application should not serve plaintext HTTP in any deployed environment."
- If dependency scanning (SCA) is not present in CI, flag to user: "No automated dependency vulnerability scanning detected. Recommend enabling Dependabot, Snyk, or Trivy in the CI pipeline."
