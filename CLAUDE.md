# Coding Agent Standards

These rules apply to all coding work across the project. Follow them in every session.

## Commit Discipline

- Atomic commits: one logical change per commit.
- Commit messages: imperative mood, under 72 chars for the subject line. Body explains WHY, not WHAT.
- Always commit and push before marking work as complete. Unpushed work does not count.
- Never force-push to main/master. Feature branches only.
- Never skip pre-commit hooks (`--no-verify`). If a hook fails, fix the issue.

## Branch Naming

- Feature: `feature/<issue-number>-<short-description>`
- Bugfix: `fix/<issue-number>-<short-description>`
- Chore: `chore/<short-description>`
- Always branch from the latest main. Rebase before PR if behind.

## Pull Requests

- Title: short, under 70 characters.
- Body: summary of changes, test plan, and link to the GitHub issue.
- One PR per issue. Do not bundle unrelated changes.
- All CI checks must pass before requesting review.

## Tool Usage

### RTK (Rust Token Killer)

RTK runs transparently via a PreToolUse hook. You do not need to invoke it manually.
If `rtk --version` fails or returns unexpected output, stop and report.

### LSP

Use LSP for code navigation: go-to-definition, find-references, hover info.
Enabled via `ENABLE_LSP_TOOL=1`. Available for Python (pyright), Go (gopls), TypeScript.

### jcodemunch

Use for structural code queries: symbol search, file outlines, blast radius analysis.
Must run `index_repo` before queries return results for a repo.

### jdocmunch

Use for navigating large markdown docs by section rather than reading entire files.
Must index the repo or directory first via `doc_index_repo` or `index_local`.

## Memory and Knowledge Graphs

Before asking clarifying questions, check these sources in order:

### claude-mem (cross-session memory)

Search prior session context via the `mcp__plugin_claude-mem_mcp-search__*` tools:
1. `search(query)` → get index with IDs
2. `timeline(anchor=ID)` → surrounding context
3. `get_observations([IDs])` → full details

### graphify knowledge graphs

Query in this order (most specific to broadest):
1. **Project-local**: `./graphify-out/` — repo-specific code and docs
2. **Wiki graph**: `~/workspace/data-worklog/graphify-out/` — Partseeker docs, ADRs, domain knowledge
3. **Supergraph**: `~/workspace/.partseeker-supergraph/` — cross-repo relationships

Start with `GRAPH_REPORT.md` for god nodes and community structure.

### When to ask the user

Only ask if the answer is genuinely not in memory or graphs. When you do ask, state what you already checked.

## Code Standards

Follow the standards in `core/docs/standards/`. Key files:
- `code-quality.md` — structure, naming, dependencies
- `testing.md` — test coverage requirements
- `error-handling.md` — logging and error patterns
- `security.md` — input validation, auth, TLS
- `workflow-discipline.md` — how to approach work

Do not deviate from these standards without explicit user approval.

## Completion Protocol

When your task is done:
1. All tests pass.
2. All changes are committed and pushed.
3. Write a completion report to `completions/` in the worklog repo (if applicable).
4. Include: what was done, evidence of verification, any architectural escalations.

## Project-Specific Context

Load repo-specific overrides, paths, and conventions:

```
@.claude/repo-context.md
```

---

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
- State assumptions before acting. When ambiguous, present interpretations and ask — do not guess silently.
- Minimum code that solves the problem. No speculative features, premature abstractions, or error handling for impossible scenarios.
- Touch only what the task requires. Do not improve, refactor, or restyle adjacent code you were not asked to change.
- Close the loop: define verifiable success criteria before starting, run tests to confirm each step, do not mark done until verified.

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

### MCP Server Safety

- All tools and integrations are read-only by default. Write, mutate, or destroy operations require explicit user approval and must be logged (who approved, what changed, which environment).
- AWS MCP must be read-only. Propose all changes as CloudFormation templates with a cost summary.
- Never create, modify, or delete cloud resources without explicit user confirmation. Describe what will happen, state the estimated cost, and confirm the target environment (dev/staging/prod) first.
- Cheapest viable option. Use the smallest resource that works. Flag costs before the user deploys anything.
- Tag every cloud resource and IaC template: `Project`, `Environment`, `CreatedBy: claude-code`.
- Read `docs/standards/mcp-safety.md` before any MCP server interaction that involves write operations.

## PR Merge Checklist

Before accepting any PR, run all three:

1. `/review-pr` — code review
2. `/security-review` — security review
3. `/simplify` — code simplifier

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
- [MCP Server Safety](docs/standards/mcp-safety.md)
- [Workflow Discipline](docs/standards/workflow-discipline.md)
