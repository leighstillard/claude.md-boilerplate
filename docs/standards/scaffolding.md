# Scaffolding & External Dependencies

This file covers the grey zone: things Claude Code can scaffold or generate, but the user must operationalise. Every item here follows the SCAFFOLD / FLAG pattern.

## Containerisation

- **SCAFFOLD**: Generate `Dockerfile` with multi-stage build, non-root user, minimal base image (e.g., `distroless` or `alpine`), explicit `COPY` of only needed artifacts, and `HEALTHCHECK` instruction.
- **SCAFFOLD**: Generate `docker-compose.yml` for local development with service dependencies (database, cache, message queue).
- **FLAG**: Container registry, image scanning, and orchestration (ECS, EKS, etc.) must be configured by the user.
- **FLAG**: Production container runtime security (read-only filesystem, dropped capabilities, resource limits) must be configured at the orchestration layer.

## CI/CD Configuration

- **SCAFFOLD**: Generate CI workflow files (`.github/workflows/*.yml` or `.gitlab-ci.yml`) with:
  - Lint and format check stage
  - Unit and integration test stage
  - SAST step (CodeQL or Semgrep config)
  - SCA step (Dependabot config, Snyk, or Trivy)
  - Build and push stage (image build if containerised)
  - Deploy stage with environment separation
- **SCAFFOLD**: Generate `.pre-commit-config.yaml` with hooks for `detect-secrets`, linting, formatting.
- **FLAG**: CI runner infrastructure must be provisioned by the user (GitHub-hosted runners, self-hosted, etc.).
- **FLAG**: Secret variables for CI (AWS credentials, registry tokens, deploy keys) must be configured in the CI platform settings.
- **FLAG**: Branch protection rules, required reviewers, and merge gates must be configured by the user.

## Infrastructure as Code

- **SCAFFOLD**: Generate Terraform or CDK templates when asked, following these practices:
  - Modular structure (separate modules for networking, compute, data, monitoring)
  - Explicit variable definitions with types and descriptions
  - Output values for cross-module references
  - Remote state backend configuration (template only — actual backend must be provisioned)
  - Environment-specific variable files (dev.tfvars, staging.tfvars, prod.tfvars)
- **FLAG**: Terraform state backend (S3 bucket, DynamoDB lock table) must be provisioned by the user.
- **FLAG**: IAM credentials for Terraform execution must be configured by the user.
- **FLAG**: The user must review and apply all infrastructure changes. Claude Code does not run `terraform apply` against real infrastructure without explicit instruction.

## Runbooks & Operational Documentation

- **SCAFFOLD**: Generate runbook templates with this structure:
  - **Alert name and severity**
  - **What this alert means** (plain language)
  - **First three things to check** (specific commands or dashboard links as placeholders)
  - **Common root causes** (based on known failure modes)
  - **Escalation path** (placeholder for team-specific contacts)
  - **Recovery steps** (specific to the failure mode)
- **FLAG**: Operational teams must fill in environment-specific details: actual dashboard URLs, actual escalation contacts, actual recovery commands.
- **FLAG**: Runbooks must be reviewed and tested against real incidents. Generated templates are starting points, not finished documents.

## Monitoring & Alerting Configuration

- **SCAFFOLD**: Generate CloudWatch alarm definitions or Grafana dashboard JSON when asked. Include:
  - Request rate and error rate panels
  - Latency percentiles (p50, p95, p99)
  - Dependency health status
  - Resource saturation (CPU, memory, connections)
- **FLAG**: The monitoring platform (CloudWatch, Grafana, Datadog) must be provisioned by the user.
- **FLAG**: Alerting notification channels (Slack, PagerDuty, email) must be configured by the user.
- **FLAG**: Alert thresholds should be reviewed against actual baseline traffic after initial deployment.

## Database Infrastructure

- **SCAFFOLD**: Generate migration files, seed scripts, and schema definitions.
- **SCAFFOLD**: Generate connection configuration with pooling, retry, and timeout settings.
- **FLAG**: Database instance provisioning (RDS, Aurora, etc.) must be done by the user.
- **FLAG**: Backup policies, maintenance windows, and failover configuration are infrastructure concerns.
- **FLAG**: Production database credentials must be provisioned in the secrets manager by the user.
