# API Design

> **Last reviewed:** March 2026. If this date is more than 6 months ago, ask Claude to review this file against current best practices before relying on it.

## DO

- Version APIs from the start. Use URL path versioning (`/api/v1/`). Plan for v2 coexistence.
- Validate input at the API boundary with typed request schemas. Return field-level error details on validation failure.
- Use a consistent error response schema across all endpoints:
  ```json
  {
    "error": {
      "code": "VALIDATION_ERROR",
      "message": "Human-readable description",
      "details": [{"field": "email", "issue": "invalid format"}],
      "reference": "correlation-id"
    }
  }
  ```
- Paginate all list endpoints. Use cursor-based pagination for large or frequently-updated datasets. Include `limit` with a sensible default and enforced maximum.
- Use idempotency keys for all state-changing operations (POST, PUT). Accept a client-supplied idempotency key and return the cached result on retry.
- Implement rate limiting with clear response headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`. Return 429 with `Retry-After`.
- Generate OpenAPI/Swagger spec from code annotations or decorators. The spec must be the source of truth, not a manually maintained document.
- Use separate request/response DTOs from internal domain models. API shapes should not leak internal structure.
- Return appropriate HTTP status codes: 201 for creation, 204 for no-content success, 409 for conflicts, 422 for semantic validation errors.
- Document breaking vs non-breaking changes. Adding a field is non-breaking. Removing or renaming a field is breaking.

## DO NOT

- Do not return 200 for everything. Status codes carry meaning for clients and monitoring.
- Do not expose internal database IDs, auto-increment values, or sequential identifiers in API responses. Use UUIDs.
- Do not return unbounded lists. If there is no `limit` parameter, add one.
- Do not expose stack traces, internal paths, or dependency names in error responses.
- Do not accept nested objects deeper than 3 levels without strong justification. Deep nesting creates validation complexity and tight coupling.
- Do not version by header or query parameter unless there is a specific technical reason. URL versioning is simplest to reason about.
- Do not create endpoints that combine unrelated operations. One endpoint, one responsibility.

## REQUIRE (external dependency)

- If no API gateway is configured (AWS API Gateway, Kong, etc.), flag to user: "Rate limiting and auth are implemented at the application layer, but an API gateway should be configured for production to provide an additional layer of throttling, TLS termination, and access control."
