# Code Quality

## DO

- Follow a consistent project structure. Group by domain/feature, not by technical layer. Prefer `internal/users/handler.go` over `handlers/users.go`.
- Separate concerns clearly: transport (HTTP handlers), business logic (services), data access (repositories). Each layer depends inward, never outward.
- Name things for clarity. Functions describe what they do. Variables describe what they hold. Packages describe what they contain. Avoid abbreviations that aren't universally understood.
- Keep functions short and single-purpose. If a function needs a comment explaining what it does, it should probably be split.
- Commit linting and formatting config: `golangci-lint` for Go, `ruff`/`mypy` for Python, `eslint`/`prettier` for TS. Enforce in CI.
- Pin all dependencies to exact versions via lockfiles (`go.sum`, `poetry.lock`, `package-lock.json`). Review dependency update diffs deliberately.
- Externalise all configuration. No hardcoded URLs, ports, timeouts, or feature flags. Use environment variables with typed defaults.
- Make configuration environment-aware (dev, staging, prod) without code changes. Same binary, different config.
- Write meaningful README content: how to set up locally, how to run tests, architecture overview with component relationships, deployment notes.
- Use interfaces/abstractions at system boundaries (database, external APIs, message queues). Concrete implementations are injected, not hardcoded.

## DO NOT

- Do not use magic numbers or strings. Define constants with descriptive names.
- Do not use global mutable state. Pass dependencies explicitly via constructors or function parameters.
- Do not write functions longer than ~50 lines without strong justification. Long functions are hard to test and hard to understand.
- Do not leave commented-out code in the codebase. Use version control to preserve history.
- Do not suppress linter warnings without a comment explaining why.
- Do not introduce dependencies for trivial functionality. Evaluate whether the standard library handles it first.
- Do not mix business logic with transport concerns. An HTTP handler should parse the request, call a service, and format the response. Nothing more.
