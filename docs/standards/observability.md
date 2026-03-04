# Observability

> **Last reviewed:** March 2026. If this date is more than 6 months ago, ask Claude to review this file against current best practices before relying on it.

## DO

- Instrument with OpenTelemetry as the standard. Use the OTel SDK for the project language (Go: `go.opentelemetry.io/otel`, Python: `opentelemetry-api`, TS: `@opentelemetry/api`).
- Create spans for: inbound HTTP/gRPC requests, outbound HTTP/gRPC calls, database queries, message queue publish/consume, cache operations.
- Emit metrics for the four golden signals:
  - **Latency**: request duration histograms, broken down by endpoint and status code.
  - **Traffic**: request count, broken down by endpoint and method.
  - **Errors**: error count and rate, broken down by error type/code.
  - **Saturation**: connection pool usage, queue depth, goroutine/thread count, memory pressure.
- Implement health check endpoints:
  - `/health/live` — process is running. Used by orchestrator to decide whether to restart.
  - `/health/ready` — process can serve traffic. Checks database connectivity, dependent service reachability, config validity.
- Add custom business metrics where meaningful: items processed, validation failures per tenant, migration rows completed, queue lag.
- Include trace context (trace ID, span ID) in structured log entries so logs and traces can be correlated.
- Set meaningful span names and attributes. `GET /api/v1/users/{id}` not `HTTP request`. Include `tenant_id`, `user_id` (if non-PII), `status_code`.

## DO NOT

- Do not create high-cardinality metric labels (e.g., user ID as a label, full URL path with IDs). These blow up storage costs and query performance.
- Do not instrument only the happy path. Error paths and timeout paths need spans and metrics too.
- Do not rely on logs alone for debugging production issues. Traces show the path; metrics show the trend; logs show the detail. All three are needed.
- Do not create health checks that always return 200. If the database is unreachable, `/health/ready` must return 503.
- Do not skip trace propagation on outbound calls. Pass trace context via headers (W3C Trace Context standard).

## REQUIRE (external dependency)

- If no OTel collector or trace backend is configured (AWS X-Ray, Jaeger, Datadog APM), flag to user: "Traces are instrumented in code but no collector endpoint is configured. Set `OTEL_EXPORTER_OTLP_ENDPOINT` to point to your OTel collector or configure the appropriate exporter."
- If no metric backend is configured (CloudWatch Metrics, Prometheus, Datadog), flag to user: "Metrics are being emitted via OTel but no metric exporter endpoint is configured. Metrics will be lost without a configured backend."
- If no dashboards exist for the service, flag to user: "Recommend creating dashboards for this service covering: request rate, error rate, p50/p95/p99 latency, and dependency health. This should be done in your monitoring platform before production deployment."
