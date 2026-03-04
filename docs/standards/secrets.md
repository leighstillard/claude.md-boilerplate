# Secrets Management

> **Last reviewed:** March 2026. If this date is more than 6 months ago, ask Claude to review this file against current best practices before relying on it.

## DO

- Read all secrets from environment variables or a config abstraction layer. Centralise this in a single config module.
- Create a config module that provides typed access to all configuration, with clear error messages when required values are missing. Fail fast on startup if critical config is absent.
- Commit `.env.example` with placeholder values and descriptive comments. Document every expected variable.
- Add `.env` to `.gitignore`. Verify it is listed before first commit.
- Generate `.pre-commit-config.yaml` with `detect-secrets` (or equivalent) configured and ready to use.
- Use short-lived tokens (OAuth2 client credentials, AWS IAM roles, STS assume-role) for service-to-service auth. Prefer these over long-lived API keys.
- Scope API keys and service credentials to minimum required permissions.
- Implement secret redaction in the logging middleware. Scan log output for patterns matching tokens, keys, and connection strings.

## DO NOT

- Do not hardcode secrets, tokens, keys, passwords, or connection strings anywhere. Not in code, not in comments, not in test files, not in documentation.
- Do not commit `.env` files, `credentials.json`, private keys, or any file containing real secrets.
- Do not log secrets. Not at any log level. Not "temporarily for debugging."
- Do not pass secrets as command-line arguments (they appear in process listings).
- Do not embed secrets in Docker images or build artifacts. Use runtime injection.
- Do not use the same secrets across environments (dev, staging, prod).
- Do not store secrets in source control even if encrypted, unless using a purpose-built tool (SOPS, sealed-secrets). Prefer external secret managers.

## REQUIRE (external dependency)

- If no secrets manager is configured (AWS Secrets Manager, Vault, etc.), flag to user: "No secrets manager detected. Secrets are currently read from environment variables only. For production, configure AWS Secrets Manager or equivalent and update the config module to read from it."
- If no pre-commit hooks are installed in the local environment, flag to user: "Pre-commit hooks for secret detection are configured but not installed. Run `pre-commit install` to activate."
- If no secret rotation policy exists, flag to user: "Secrets should have automated rotation configured in the secrets manager. Current code supports rotation (reads on each use, no caching of raw secret values) but rotation must be enabled externally."
