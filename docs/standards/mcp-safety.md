# MCP Server Safety

> **Last reviewed:** March 2026. If this date is more than 6 months ago, ask Claude to review this file against current best practices before relying on it.

MCP servers give Claude Code the ability to create, modify, and destroy real infrastructure, spend real money, and affect production systems. This file defines guardrails for safe MCP usage.

**General principle: MCP servers should be configured with the minimum permissions required. Read-only by default. Write access only when explicitly needed and with human approval.**

## AWS MCP Server

**The AWS MCP server should be connected with read-only IAM credentials only.** All infrastructure changes should be proposed as CloudFormation templates that the user reviews and deploys through their own pipeline. This prevents accidental resource creation, surprise bills, and unaudited changes.

### The Pattern

1. **Discover** — Use AWS MCP (read-only) to inspect current infrastructure, understand existing resources, check configurations, and identify gaps.
2. **Propose** — Generate a CloudFormation template for the required changes. Include all resources, parameters, and outputs.
3. **Estimate** — Produce a cost summary for every proposed change before the user deploys anything (see Cost Summary below).
4. **Hand off** — Present the CloudFormation template and cost summary to the user. They review, adjust, and deploy through their own pipeline.

### DO

- Connect AWS MCP with read-only IAM credentials (e.g., `ReadOnlyAccess` managed policy or a custom read-only role).
- Use AWS MCP to inspect existing resources, describe configurations, list services, check IAM policies, and review security groups.
- Propose all infrastructure changes as CloudFormation templates. Generate complete, deployable templates with parameters, sensible defaults, and descriptive comments.
- Produce a cost summary for every CloudFormation template (see format below).
- Tag every resource in generated CloudFormation templates with at minimum: `project`, `environment`, `created-by`, and `cost-centre` (or your organisation's equivalent).
- Prefer serverless and on-demand resources (Lambda, Fargate, Aurora Serverless) over provisioned capacity in dev/test environments.
- Use the free tier or smallest available option for dev/test resources unless the user explicitly specifies otherwise.
- Include `DeletionPolicy` and `UpdateReplacePolicy` on stateful resources (databases, S3 buckets, EFS).

### DO NOT

- Do not use AWS MCP to create, modify, or delete resources directly. All mutations go through CloudFormation templates.
- Do not connect AWS MCP with write permissions (`AdministratorAccess`, `PowerUserAccess`, or any policy allowing `Create*`, `Delete*`, `Modify*`, `Put*` actions).
- Do not generate CloudFormation templates with public access patterns (public S3 buckets, publicly accessible RDS, security groups open to 0.0.0.0/0) without explicit user approval.
- Do not propose expensive instance families (p4, p5, g5, x2, etc.) without explicit user request and a cost flag.
- Do not propose multi-AZ, multi-region, or high-availability configurations for dev/test environments unless explicitly asked.
- Do not assume the correct AWS account or region. Confirm with the user if there is any ambiguity.

### REQUIRE

- If no AWS budget alarm is detected in the account, flag to user: "No budget alarm detected. Recommend configuring an AWS Budget with alert thresholds before deploying any CloudFormation stacks."
- If the AWS MCP credentials have write permissions, flag to user: "AWS MCP is connected with write permissions. Recommend replacing with a read-only IAM role. All changes should be proposed via CloudFormation templates."

### Cost Summary Format

Every CloudFormation template must be accompanied by a cost summary in this format:

```
## Cost Estimate: [Stack Name]

| Resource | Type | Configuration | Estimated Monthly Cost |
|---|---|---|---|
| AppDatabase | RDS (Aurora Serverless v2) | 0.5–2 ACU, 20GB storage | $45–$180 |
| AppQueue | SQS (Standard) | ~100k messages/month | <$1 |
| ... | ... | ... | ... |

**Estimated Total: $X–$Y/month**

### Cost Warnings
- [Any resources from the cost traps list below]
- [Any resources with usage-dependent pricing that could scale unexpectedly]

### Assumptions
- [Traffic/usage assumptions the estimate is based on]
- [Region: ap-southeast-2 or wherever]
```

Cost estimates do not need to be exact. Ballpark ranges are fine. The goal is to prevent surprise bills, not replace the AWS Pricing Calculator.

### Common Cost Traps

These are resources that generate surprisingly high bills. Flag in the cost summary whenever a template includes them:

- NAT Gateways (~$32/month each + data processing charges)
- Elastic IPs not attached to running instances ($3.60/month each)
- RDS Multi-AZ deployments (double the single-AZ cost)
- EBS volumes left attached to stopped instances (storage charges continue)
- CloudWatch Logs with no retention policy (storage grows indefinitely)
- Kinesis Data Streams with over-provisioned shard count
- Elasticsearch/OpenSearch domains (minimum ~$80/month for even small instances)
- Load Balancers with no targets (still incur hourly charges)
- Large EBS gp3 volumes with provisioned IOPS/throughput beyond defaults
- Secrets Manager secrets ($0.40/secret/month — adds up with many microservices)
- KMS keys ($1/key/month — also adds up)
- Config rules ($1/rule evaluation — can be significant with many rules and frequent changes)


## GitHub MCP Server

### DO

- Use a Personal Access Token scoped to only the repositories needed. Prefer fine-grained tokens over classic tokens.
- Confirm the target repository and branch before any write operation (creating PRs, pushing commits, modifying branch protection).
- Create feature branches for changes. Never push directly to main/master.
- Use the read-only toolset by default. Only enable write toolsets when actively needed.
- Use lockdown mode for public repositories to prevent prompt injection from untrusted contributors.

### DO NOT

- Do not force-push to any branch without explicit user approval.
- Do not delete branches without confirming with the user.
- Do not merge pull requests without user review and approval.
- Do not modify branch protection rules without explicit user approval. These are security controls.
- Do not create repository secrets via MCP — secrets should be managed through a dedicated secrets workflow with proper audit trails.
- Do not modify GitHub Actions workflows without user review. Workflow changes can execute arbitrary code in CI.
- Do not expose sensitive data in PR titles, descriptions, commit messages, or issue bodies.


## Terraform MCP Server

### DO

- Always run `plan` before `apply`. Show the plan output to the user and get explicit approval before applying.
- Use workspaces to separate environments. Confirm the target workspace before any operation.
- Prefer read-only operations (registry lookups, documentation queries, state inspection) as the default mode of use.
- Keep `ENABLE_TF_OPERATIONS` disabled unless actively applying changes.

### DO NOT

- Do not run `terraform apply` without showing the plan to the user first and getting explicit approval.
- Do not run `terraform destroy` without double-confirming with the user. State what will be destroyed and ask for explicit confirmation.
- Do not modify Terraform state directly (state mv, state rm, import) without explicit user instruction and approval. State corruption can be irreversible.
- Do not apply changes to production workspaces without extra confirmation: "This will modify PRODUCTION infrastructure. Please confirm."
- Do not assume the correct Terraform workspace. Confirm with the user before any state-modifying operation.


## PagerDuty MCP Server

### DO

- Use read-only mode by default (`--enable-write-tools` should be off unless actively managing incidents).
- Confirm before creating incidents — false incidents cause alert fatigue and erode trust in the alerting system.
- When resolving or acknowledging incidents, confirm the incident ID and summary with the user before taking action.

### DO NOT

- Do not acknowledge or resolve incidents without explicit user approval. These are operational decisions with on-call implications.
- Do not create test incidents in production PagerDuty services. Use a dedicated test service if available.
- Do not modify escalation policies or on-call schedules without explicit user approval. These affect real people's lives (literally — someone's phone rings at 3 AM).
- Do not bulk-acknowledge or bulk-resolve incidents. Handle them individually with user confirmation.


## Datadog / Grafana MCP Servers

### DO

- Use primarily for read operations: querying metrics, searching logs, exploring traces.
- Confirm before creating or modifying monitors/alerts — bad alert thresholds cause alert fatigue or missed incidents.
- Verify the correct Datadog site/region before connecting.

### DO NOT

- Do not create monitors that alert on-call teams without user review of the alert conditions and thresholds.
- Do not delete or mute existing monitors without explicit user approval.
- Do not modify dashboards that are shared with or owned by other teams without flagging this to the user.


## SonarQube / Snyk MCP Servers

These are primarily read-only analysis tools and carry lower risk. Standard precautions:

### DO

- Use for querying issues, checking quality gates, and analysing code snippets.
- Review findings with the user before marking issues as false positives or accepted.

### DO NOT

- Do not bulk-dismiss or bulk-accept security findings without user review of each finding.
- Do not change issue statuses (accept, false positive) without explaining the rationale to the user and getting approval.


## General MCP Safety Rules

- **AWS is read-only.** Use AWS MCP for inspection and discovery only. All changes are proposed as CloudFormation templates with cost summaries. No exceptions.
- **Default to read-only.** If a server supports read-only mode, use it unless a write operation is explicitly needed.
- **Confirm before mutating.** Any operation that creates, modifies, or deletes resources requires explicit user confirmation. Repeat back what will happen and wait for approval.
- **Cheapest viable option.** When proposing cloud resources in CloudFormation or IaC templates, default to the smallest, cheapest option that meets the stated requirement. Include a cost summary.
- **No assumptions about environment.** Always confirm whether an operation targets dev, staging, or production. If unclear, ask.
- **Audit trail.** Tag or label all resources in generated templates so they can be identified and cleaned up. Include `created-by: claude-code` or equivalent.
- **Principle of least privilege.** MCP server credentials should have the minimum permissions needed. Flag if credentials appear to have overly broad access.
