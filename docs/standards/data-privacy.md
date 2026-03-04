# Data Privacy & Tenant Isolation

## DO

- Scope every data query by tenant ID. Implement this at the repository/data access layer so individual queries cannot bypass it.
- Use a middleware or context pattern to extract and propagate tenant ID from the authenticated session. Every downstream call inherits tenant context.
- Classify data fields at schema design time. Use naming conventions (`_pii`, `_sensitive`) or annotations/comments to mark fields containing personal or sensitive data.
- Build PII masking utility functions. Use them in every logging and serialisation path. Mask by default, reveal only when explicitly needed.
- Implement soft delete for all user-facing data. Include `deleted_at` timestamp and `deleted_by` reference.
- Add retention policy hooks: code-level support for automated purge/archive of data past its retention window. The scheduling is external; the logic must exist in code.
- Create read-only views or DTOs that exclude sensitive fields for lower-privilege consumers or external APIs.
- Include consent tracking fields in user data models where regulatory requirements apply (GDPR consent, marketing opt-in).
- Separate PII from operational data where practical. Different tables, different access controls.

## DO NOT

- Do not make tenant scoping optional or configurable per query. It is always on.
- Do not log PII fields. Not in debug logs, not in error logs, not in audit logs unless the audit log is specifically designed and access-controlled for it.
- Do not return tenant IDs or other tenant-identifying data in error messages to end users.
- Do not allow cross-tenant joins or queries in any application code path. If analytics requires cross-tenant aggregation, it must be a separate, access-controlled analytical path.
- Do not hard delete user data unless there is an explicit, documented policy requiring it (e.g., right to be forgotten implementation). Even then, audit log the deletion.
- Do not expose internal database IDs in APIs. Use UUIDs or opaque identifiers.

## REQUIRE (external dependency)

- If no tenant context is available in the request pipeline (e.g., no identity provider is configured), flag to user: "Tenant isolation requires authenticated tenant context. Identity provider and auth middleware must be configured."
- If the database does not support row-level security or equivalent, flag to user: "Application-layer tenant filtering is implemented, but database-level row-level security should be configured as a defence-in-depth measure."
- If data classification requirements exist (GDPR, PCI-DSS, APRA CPS 234), flag to user: "Data classification policy must be defined at the organisational level. Current schema annotations assume [default classification]. Review and adjust."
