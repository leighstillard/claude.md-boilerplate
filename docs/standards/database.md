# Database Practices

## DO

- Use versioned, reversible migrations. Every migration must have an `up` and `down` path. Name migrations with timestamps, not sequence numbers.
- Make schema changes backwards-compatible by default. Add columns as nullable or with defaults. Remove columns in a subsequent release after code no longer references them.
- Create indexes intentionally. Comment each index with the query pattern it supports. Review unused indexes periodically.
- Configure connection pooling explicitly. Set minimum, maximum, and idle timeout values based on expected load. Do not rely on driver defaults.
- Enable slow query logging. Set a threshold appropriate for the application (e.g., 100ms for OLTP, 5s for batch). Log the query, duration, and caller context.
- Choose transaction isolation levels explicitly. Understand the trade-offs: `READ COMMITTED` is the minimum for most use cases; `SERIALIZABLE` for operations requiring strict consistency.
- Use database-level constraints (NOT NULL, UNIQUE, FOREIGN KEY, CHECK) in addition to application-level validation. The database is the last line of defence.
- Scope all queries by tenant. Implement this in the repository layer so it cannot be bypassed by individual queries.
- Use parameterised queries exclusively. No string interpolation, no template literals, no `fmt.Sprintf` for SQL.
- Design for connection failure. Implement retry with backoff for transient database errors. Distinguish transient from permanent failures.

## DO NOT

- Do not write migrations that cannot be reversed. If a migration is truly one-way (e.g., dropping a column with data loss), document this explicitly and require manual approval.
- Do not run DDL and DML in the same transaction unless the database supports it cleanly (Postgres does, MySQL has limitations).
- Do not use `SELECT *`. Specify columns explicitly. This prevents breakage when columns are added and makes query intent clear.
- Do not create indexes on every column speculatively. Each index has a write-cost trade-off.
- Do not use ORM lazy loading in request handlers. Use eager loading or explicit joins to prevent N+1 queries.
- Do not store secrets (API keys, tokens) in the database without encryption. Use application-level encryption with keys from the secrets manager.
- Do not connect to production databases from local development environments. Use isolated dev/staging databases.

## REQUIRE (external dependency)

- If no database backup strategy is configured, flag to user: "Automated backups must be configured at the infrastructure level (RDS automated backups, Aurora backups, or equivalent). Code supports point-in-time recovery patterns but backups must be enabled externally."
- If row-level security is available (Postgres RLS), flag to user: "Row-level security policies should be configured as a defence-in-depth measure alongside application-layer tenant filtering. This requires database-level configuration."
- If no read replica is configured for read-heavy workloads, flag to user: "Read-heavy query patterns detected. Consider configuring a read replica and routing read queries to it."
