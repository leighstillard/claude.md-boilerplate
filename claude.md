# Project Standards

## Language & Technology Defaults

- Prefer Go. Python 3 acceptable with strict type hints (mypy strict). TypeScript strict mode for frontend.
- When in doubt, choose the strongest type system available.
- AWS serverless over self-managed (SQS over Kafka, Aurora Serverless over self-hosted Postgres, Secrets Manager over Vault).
- Open source over proprietary at comparable quality.
- OpenTelemetry for observability. Structured JSON logging only.

## Behavioural Rules

- Propose test cases before writing implementation. Get approval.
- Before modifying existing code, run existing tests to confirm a passing baseline.
- When asked to "make it work" or "just get it done", do not skip error handling, logging, validation, or tests.
- If a required external dependency is missing (secrets manager, auth provider, log collector, CI pipeline), stop and flag it to the user. Do not work around it with placeholders or local substitutes.
- When generating files in a specific domain (API, database, auth), read the relevant pillar doc in `docs/standards/` first.
- Use conventional commits format for commit messages.
- Prefer small, reviewable changesets over large monolithic changes.

## Critical Rules (always apply)

### Security

- Never hardcode secrets, tokens, keys, or connection strings. No exceptions. No "temporary" ones.
- Parameterised queries only. No string concatenation for SQL or NoSQL queries.
- Input validation at every boundary (API, queue consumer, file ingestion). Allowlists over denylists.
- Centralised auth middleware. Never check permissions inside individual handlers.

### Data Privacy

- Every data access must be scoped by tenant. No unscoped queries, no optional tenant filters.
- PII fields annotated in schema. PII masked in all log output.
- Soft delete with retention hooks. No hard deletes of user data without explicit policy.

### Testing

- No feature code merged without tests. Tests are not optional under time pressure.
- Every test suite must include: happy path, edge cases, error/failure conditions, auth enforcement, tenant isolation.
- Specifically test that user A cannot access user B's data.

### Error Handling

- No swallowed exceptions. Every catch block must handle, log, or re-throw.
- User-facing errors: generic, safe, no internal details. Internal errors: specific, contextual, actionable.
- Every log entry: structured JSON, correlation ID, timestamp, service name.

## Detailed Standards

Read the relevant file before working in that domain:

- [Security](docs/standards/security.md)
- [Data Privacy & Tenant Isolation](docs/standards/data-privacy.md)
- [Secrets Management](docs/standards/secrets.md)
- [Testing](docs/standards/testing.md)
- [Error Handling & Logging](docs/standards/error-handling.md)
- [Observability](docs/standards/observability.md)
- [API Design](docs/standards/api-design.md)
- [Database](docs/standards/database.md)
- [Code Quality](docs/standards/code-quality.md)
- [Scaffolding & External Dependencies](docs/standards/scaffolding.md)
