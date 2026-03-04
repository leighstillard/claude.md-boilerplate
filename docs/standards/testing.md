# Testing Standards

> **Last reviewed:** March 2026. If this date is more than 6 months ago, ask Claude to review this file against current best practices before relying on it.

## DO

- Write test cases before implementation. Propose them, get agreement, then build.
- Create test files alongside every module. Same directory, predictable naming (`foo_test.go`, `test_foo.py`, `foo.test.ts`).
- Cover these categories for every feature:
  - **Happy path**: expected inputs produce expected outputs.
  - **Edge cases**: empty inputs, max-length inputs, boundary values, unicode, special characters.
  - **Error conditions**: invalid input, missing required fields, downstream failures, timeouts.
  - **Auth enforcement**: unauthenticated requests are rejected. Wrong-role requests are rejected.
  - **Tenant isolation**: user A's request cannot return user B's data. Test this explicitly, not implicitly.
- Write integration tests for cross-component data flows. Test the actual path from API to database and back.
- Write contract tests or schema validation for service-to-service interfaces. If service B changes its response shape, service A's tests should break.
- Use test fixtures and factories that produce realistic data. Avoid trivial stubs that mask real-world behaviour.
- Add benchmark tests for performance-sensitive paths (hot loops, batch processing, query-heavy endpoints).
- Test error logging: verify that when errors occur, the correct log level, correlation ID, and context are emitted.
- Test that PII masking works: log output from test runs should not contain sensitive field values.
- Run the existing test suite before making changes. Establish the passing baseline. If tests are already failing, flag to user before proceeding.

## DO NOT

- Do not write tests that only verify the happy path and call coverage "complete."
- Do not mock so aggressively that the test validates the mock, not the code. Mock external boundaries (HTTP clients, databases), not internal logic.
- Do not write tests with hardcoded secrets, real API keys, or real user data. Use generated fixtures.
- Do not skip tests to meet a deadline. If asked to, push back and explain the risk.
- Do not write flaky tests. If a test depends on timing, order, or external state, fix the design or flag it.
- Do not treat test code as second-class. Same linting, same formatting, same review standards as production code.
- Do not ignore test failures in CI. A failing test suite is a blocker, not a warning.

## Gap Analysis

When reviewing an existing test suite, actively look for these common gaps:

- **Missing negative paths**: what happens when required fields are absent? When types are wrong?
- **Missing concurrency tests**: what happens when two requests modify the same resource simultaneously?
- **Missing tenant boundary tests**: can a valid but wrong tenant ID retrieve data?
- **Missing downstream failure tests**: what happens when the database is slow? When an external API returns 500? When a message queue is unavailable?
- **Missing idempotency tests**: if the same request is sent twice, does the system handle it correctly?
- **Missing rollback tests**: if a multi-step operation fails partway through, is the system in a consistent state?

Flag any gaps found to the user with specific recommendations for tests to add.
