# Error Handling & Logging

> **Last reviewed:** March 2026. If this date is more than 6 months ago, ask Claude to review this file against current best practices before relying on it.

## DO

- Use structured JSON logging exclusively. Every log entry must include: `timestamp`, `level`, `message`, `correlation_id`, `service_name`.
- Include `tenant_id` in log entries where tenant context exists.
- Generate a correlation ID at the ingress point (API gateway, first handler). Propagate it through all internal calls, database queries, and outbound requests via headers or context.
- Use log levels consistently and with these meanings:
  - `DEBUG`: developer context, only enabled in dev/staging. Variable values, flow tracing.
  - `INFO`: business events. Request received, operation completed, state transitions.
  - `WARN`: recoverable issues that may need attention. Retry succeeded, deprecated endpoint called, approaching rate limit.
  - `ERROR`: failures needing investigation. Request failed, dependency unavailable, data inconsistency detected.
  - `FATAL`: process cannot continue. Missing critical config, unrecoverable state, startup failure.
- Create custom error types with error codes. Map error codes to runbook entries where possible.
- Sanitise request bodies before logging. Strip or mask fields matching known PII patterns.
- Return generic error messages to users: "Something went wrong. Reference: [correlation_id]". The correlation ID lets support trace the issue internally.
- Log at the boundary where the error is handled, not at every level it passes through. Avoid duplicate log entries for the same error.

## DO NOT

- Do not swallow exceptions. Every catch/recover block must do at least one of: handle the error, log it, or re-raise/re-return it.
- Do not use bare `catch` (JS/TS), bare `except` (Python), or bare `recover` (Go) without logging context.
- Do not log stack traces to user-facing output. Stack traces go to internal logs only.
- Do not log PII, tokens, passwords, full credit card numbers, or session IDs. Not at any level.
- Do not use `fmt.Println`, `print()`, or `console.log` for production logging. Use the structured logger.
- Do not log at ERROR level for expected conditions (e.g., user not found, validation failure). These are INFO or WARN.
- Do not create log entries without correlation IDs. If correlation context is missing, that itself is a bug to flag.
- Do not log the same error at multiple layers. Catch and log at the handling layer, propagate without logging at intermediate layers.

## REQUIRE (external dependency)

- If no log aggregation service is configured (CloudWatch Logs, Datadog, ELK), flag to user: "Structured logs are being written to stdout. For production, configure a log collector to ship these to a centralised aggregation service."
- If no alerting is configured on error rates, flag to user: "Error-rate alerting should be configured in the monitoring platform. Recommended: alert when error rate exceeds [X]% over a 5-minute window."
