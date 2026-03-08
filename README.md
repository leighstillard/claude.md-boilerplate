# Claude Code Standards Template

An opinionated `claude.md` template that steers Claude Code toward production-grade software engineering practices. This has been designed to have minimal impact on context usage while still helping provide the guardrails and feedback that help build operable software. Built for teams that care about security, data privacy, observability, and testability — not just "does it run."

## The Problem

Claude Code is remarkably capable at generating functional software. But "functional" and "production-ready" are different standards. Without guidance, AI-assisted development tends to produce code that works but cuts corners on the things that matter when software runs in the real world: tenant isolation, structured logging, secret handling, negative-path testing, and graceful failure.

A `claude.md` file is Claude Code's persistent instruction set — it's loaded automatically and shapes every interaction. This repo provides a template that uses that mechanism to enforce engineering discipline by default.

## What This Is Not

**Not a compliance certification.** This template references GDPR, PCI-DSS, APRA CPS 234, and other regulations as examples of where technical controls apply. It does not make your software compliant. Compliance is an organisational, legal, and procedural exercise that requires qualified professionals. The template provides some of the technical patterns that compliance *requires*, but it is not legal or compliance advice.

**Not a guarantee.** Claude Code follows instructions probabilistically, not deterministically. These rules significantly improve the baseline — but Claude Code can and will occasionally ignore them, especially in long sessions where context pressure pushes pillar files out of effective memory. This template is a guardrail, not a safety net. You still need code review, CI checks, and security audits.

**Not one-size-fits-all.** A single-user CLI tool doesn't need tenant isolation. A static site generator doesn't need correlation IDs. A hackathon prototype doesn't need PCI-DSS considerations. If you drop this template into a project without tailoring it, Claude Code will spend effort on things that add complexity for zero value. See [Getting Started](#getting-started) below for how to tailor it to your project.

**Not a snapshot in time.** Security best practices evolve, AWS services change, new vulnerability classes emerge. This template will become outdated. Each pillar file includes a `Last reviewed` date — if that date is more than 6 months old, ask Claude to review it against current best practices before relying on it.

## Getting Started

### Step 1: Tailor to Your Project

Before you start building, run this prompt in Claude Code to strip out what doesn't apply and catch anything that's out of date:

```
Review the claude.md file and all files in docs/standards/. For each pillar file:

1. Assess whether this pillar is relevant to our project: [describe your project in 1-2 sentences, e.g. "a single-tenant CLI tool for data processing" or "a multi-tenant SaaS API handling financial data"]. Remove any pillar files that are not relevant and update the references in claude.md.

2. Review the compliance references (GDPR, PCI-DSS, APRA, SOX, etc.) and remove any that do not apply to our use case: [describe your regulatory context, e.g. "we handle no PII" or "we process payment card data for Australian customers"].

3. Check all technical recommendations against current best practices. Flag anything that appears outdated, deprecated, or superseded. Update as needed.

4. Check the MCP server recommendations are still current. Verify links, check if newer/better options exist.

5. Review the cost trap list in mcp-safety.md against current AWS pricing.

6. Summarise what you changed and why.
```

### Step 2: Track Outstanding Flags

As you build, Claude Code will flag missing external dependencies (no secrets manager configured, no log aggregation, no auth provider). These flags shouldn't be treated as one-time warnings — they're a pre-production checklist.

Ask Claude Code periodically, especially before staging or production deployment:

```
Review all REQUIRE and FLAG items across the standards pillar files. For each one, assess whether it has been addressed in this project. Produce a compliance checklist showing: addressed (with evidence), not yet addressed (with recommendation), and not applicable (with justification).
```

### Step 3: Deploy

1. Copy `claude.md` and the `docs/standards/` directory into your project root.
2. Run the tailoring prompt from Step 1.
3. Start building. The root `claude.md` loads automatically in every Claude Code session.
4. Before going to production, run the flag review prompt from Step 2.

## Design Approach

### Three-Tier Responsibility Model

Not everything that matters in software delivery can be done by Claude Code. We explicitly separated concerns into three tiers:

1. **Claude Code does it** — practices that are entirely within the code: input validation, parameterised queries, tenant-scoped data access, structured logging, test coverage. These are expressed as direct rules in the `claude.md` and pillar files.

2. **Claude Code scaffolds it, you operationalise it** — things like Dockerfiles, CI/CD workflow configs, IaC templates, and runbook templates. Claude Code can generate high-quality starting points, but the infrastructure, credentials, and runtime environment behind them are your responsibility. These follow a `SCAFFOLD / FLAG` pattern.

3. **You do it** — secrets infrastructure, CI/CD runner provisioning, observability platform setup, compliance decisions, incident response processes. Claude Code can't provision your Vault cluster or decide your data classification policy. But the template ensures Claude Code *flags* when these dependencies are missing rather than silently working around them.

### Positive and Negative Rules

Each pillar uses a `DO / DO NOT / REQUIRE` structure:

- **DO** rules tell Claude Code what good practice looks like. They're specific and actionable.
- **DO NOT** rules prevent the most common and most damaging shortcuts. These tend to be more valuable than the DOs because they block failure modes that are hard to detect in review.
- **REQUIRE** rules identify external dependencies. When Claude Code encounters a gap (no secrets manager configured, no auth provider, no log aggregation), it raises it to you instead of inventing a workaround.

### Brevity Over Completeness

The root `claude.md` is deliberately short (~65 lines). Every word in that file competes with your actual conversation for context window space. The critical rules that prevent the worst outcomes live in the root file. Detailed guidance lives in pillar files under `docs/standards/`, which Claude Code reads on demand when working in a relevant domain.

This mirrors how experienced engineers work: you carry a small set of non-negotiable principles in your head, and reference detailed standards when you're doing specialised work.

### Behavioural Rules

Beyond code standards, the template includes rules about how Claude Code should *work*:

- Propose test cases before writing implementation
- Run existing tests before modifying code to establish a baseline
- Never skip error handling, logging, or tests when asked to "just make it work"
- Flag missing dependencies instead of working around them
- Prefer small, reviewable changesets

These shape the development workflow, not just the output.

## Repository Structure

```
claude.md                            # Root file — always loaded, ~65 lines
docs/
  standards/
    security.md                      # Input validation, auth, headers, crypto
    data-privacy.md                  # Tenant isolation, PII handling, retention
    secrets.md                       # Config modules, env vars, pre-commit hooks
    testing.md                       # TDD workflow, coverage categories, gap analysis
    error-handling.md                # Structured logging, correlation IDs, log levels
    observability.md                 # OpenTelemetry, golden signals, health checks
    api-design.md                    # Versioning, pagination, error schemas
    database.md                      # Migrations, connection pooling, tenant scoping
    code-quality.md                  # Project structure, naming, dependencies
    scaffolding.md                   # Dockerfiles, CI configs, IaC, runbooks (grey zone)
    mcp-safety.md                    # MCP server guardrails, cost traps, risk warnings
    workflow-discipline.md           # Assumption surfacing, simplicity, surgical changes, close the loop
```

## Technology Opinions

The template makes deliberate technology choices to reduce ambiguity. These are aligned to my preferences and what I have found is easier to debug when vibe coding, you should change them to yours:

| Concern | Choice | Rationale |
|---|---|---|
| Primary language | Go | Strong typing, excellent concurrency, fast compilation |
| Secondary language | Python 3 (strict mypy) | Ecosystem breadth, with type safety enforced |
| Frontend | TypeScript (strict mode) | Type safety for debugging, broad ecosystem |
| Observability | OpenTelemetry | Vendor-neutral, industry standard |
| Logging | Structured JSON to stdout | Portable across any log aggregation platform |
| Cloud services | AWS serverless where setup burden is high | SQS over Kafka, Aurora Serverless over self-hosted Postgres, Secrets Manager over Vault |
| General preference | Open source over proprietary at comparable quality | Avoids lock-in, community support |

These are opinions, not universal truths. Fork and adjust to your stack.

## MCP Server Considerations

During design, we evaluated which "user must do" items could potentially be automated via MCP (Model Context Protocol) servers. The table below maps each operational concern to recommended MCP servers where they exist, and calls out where you're on your own.

> **⚠️ A word of caution.** MCP servers that can create, modify, or delete cloud resources are giving an AI agent access to real infrastructure and real money. We recommend connecting the AWS MCP server with **read-only credentials only** and routing all changes through CloudFormation templates with cost summaries — see `docs/standards/mcp-safety.md` for the full pattern. The Terraform MCP server can destroy production infrastructure with a single `apply`. The PagerDuty MCP server can wake someone up at 3 AM with a false incident. We've included a dedicated safety pillar file (`docs/standards/mcp-safety.md`) with DO/DO NOT rules for each server, including a list of common AWS cost traps and a required cost summary format. **Read it before connecting any MCP server with write access.** Configure servers with read-only permissions by default, scope credentials to minimum required access, and always require explicit confirmation before any mutating operation. The template's claude.md includes rules enforcing this, but ultimately you are responsible for the credentials you provide and the permissions they carry.

### Recommended MCP Servers

| Concern | MCP Server | Maintainer | Status | What It Covers |
|---|---|---|---|---|
| **AWS Infrastructure** | [AWS MCP Server](https://github.com/awslabs/mcp) | AWS (official) | GA | **Connect with read-only credentials only.** Use for inspecting infrastructure, checking configurations, and understanding current state. All changes should be proposed as CloudFormation templates with cost summaries. Covers 15,000+ AWS APIs. |
| **CI/CD & Source Control** | [GitHub MCP Server](https://github.com/github/github-mcp-server) | GitHub (official) | GA | Repository management, branch protection, PR automation, Actions workflow monitoring, Dependabot alerts, code scanning (CodeQL), repository secrets. Remote hosted server available. |
| **Infrastructure as Code** | [Terraform MCP Server](https://github.com/hashicorp/terraform-mcp-server) | HashiCorp (official) | Beta | Registry lookups for providers/modules, HCP Terraform workspace management (create, update, delete), plan/apply operations, private registry access. |
| **Observability — Tracing** | [Grafana Cloud Traces MCP](https://grafana.com/docs/grafana-cloud/send-data/traces/mcp-server/) | Grafana (official) | Preview | TraceQL queries, service interaction exploration, span analysis. Built into Grafana Cloud, no additional install. |
| **Observability — Full Stack** | [Datadog MCP Server](https://www.datadoghq.com/blog/datadog-remote-mcp-server/) | Datadog (official) | Preview | Logs, metrics, traces, APM, monitors, SLOs, service definitions, incident context. Remote hosted server. |
| **Observability — Metrics** | [Datadog MCP (community)](https://github.com/shelfio/datadog-mcp) | shelfio (community) | Stable | Metric queries, CI pipeline data, service definitions, team management. Self-hosted alternative if not on official preview. |
| **Code Quality — SAST** | [SonarQube MCP Server](https://github.com/SonarSource/sonarqube-mcp-server) | SonarSource (official) | GA | Issue search, quality gate status, code snippet analysis, dependency risk scanning, issue status management. Works with SonarQube Cloud and Server. |
| **Code Security — SCA** | [Snyk MCP Server](https://snyk.io/) | Snyk (official) | GA | Vulnerability scanning embedded in agentic workflows, open-source dependency risk assessment. |
| **Incident Management** | [PagerDuty MCP Server](https://github.com/PagerDuty/pagerduty-mcp-server) | PagerDuty (official) | GA | Incident listing/creation/update, on-call schedules, service management, event orchestrations. Hosted and self-hosted options. Read-only by default, write requires explicit flag. |

### Risk Warnings by Server

| MCP Server | Risk Level | Key Dangers |
|---|---|---|
| **AWS MCP** | **🟡 Medium** | Recommended as **read-only only**. All changes should be proposed as CloudFormation templates with a cost summary — never created directly via MCP. Risk drops significantly with read-only credentials, but misconfigured templates can still be costly if deployed without review. See `docs/standards/mcp-safety.md` for the full Discover → Propose → Estimate → Hand Off pattern and a list of cost traps. |
| **Terraform MCP** | **🔴 High** | `apply` modifies real infrastructure. `destroy` is irreversible. State corruption from `state mv`/`state rm` can be catastrophic. Keep `ENABLE_TF_OPERATIONS` off except when actively applying. Always show the plan and get approval before apply. |
| **GitHub MCP** | **🟡 Medium** | Can merge PRs, delete branches, modify branch protection, and change Actions workflows. Force-push can destroy commit history. Workflow changes execute arbitrary code in CI. Use fine-grained tokens and lockdown mode on public repos. |
| **PagerDuty MCP** | **🟡 Medium** | Can create incidents that page real humans at any hour. Can resolve incidents prematurely, masking real outages. Can modify on-call schedules. Use read-only mode by default. |
| **Datadog / Grafana MCP** | **🟢 Low** | Primarily read-only risk. Can create or modify monitors/alerts that cause alert fatigue or miss real incidents. Confirm alert thresholds before creating. |
| **SonarQube / Snyk MCP** | **🟢 Low** | Primarily read-only. Can dismiss or accept security findings, potentially hiding real vulnerabilities. Review each finding individually. |

### Notable Gaps — No Good MCP Server Exists

| Concern | Current State | Workaround |
|---|---|---|
| **Grafana Dashboard Management** | Grafana's MCP covers tracing only. No official MCP for creating/managing dashboards, alerting rules, or Loki log queries. | Generate dashboard JSON via Claude Code, import manually or via Grafana HTTP API. |
| **Prometheus / Metrics Querying** | Community servers exist but none with official backing or production maturity. | Use Datadog MCP if on Datadog, or query Prometheus via scripts Claude Code generates. |
| **CloudWatch Dashboards & Alarms** | Covered by the broad AWS MCP Server, but dashboard/alarm creation is just raw API calls with no guided workflow. | AWS MCP can do it, but expect to provide more detailed prompts. SCAFFOLD pattern works well here. |
| **Log Aggregation Setup** | No dedicated MCP for ELK, Loki, or CloudWatch Logs configuration. | Claude Code scaffolds log shipping configs; you deploy and configure the aggregation platform. |
| **Secrets Rotation Automation** | AWS MCP can configure Secrets Manager rotation lambdas, but it's not a guided workflow. | Claude Code can generate the rotation lambda code and Terraform for the schedule; you deploy it. |
| **GitLab CI/CD** | No official GitLab MCP server at time of writing. | Claude Code generates `.gitlab-ci.yml` files directly. Manual pipeline configuration required. |
| **Confluence / Notion (Runbooks)** | Atlassian has an official MCP for Jira and Confluence, but runbook publishing is limited. | Claude Code generates runbook markdown; you publish manually or via API. |

### What MCP Servers Can't Do (Meatspace Required)

Some things need a human making decisions, not an agent executing API calls. No MCP server will save you here:

| Concern | Why It's Human-Only |
|---|---|
| **Data classification policy** | Organisational decision about what's sensitive, what's restricted, what retention applies. Claude Code enforces the technical controls, but someone has to decide the policy. |
| **Regulatory mapping** | Deciding whether GDPR, SOX, PCI-DSS, APRA CPS 234 or other regulations apply to your system is a legal and compliance exercise. |
| **Privacy impact assessments** | Legal/compliance exercise that requires understanding of data flows, jurisdictions, and regulatory obligations. |
| **SLO/SLA definition** | Business decision about what uptime and performance guarantees you're committing to. MCP servers can monitor them once defined, but the targets are a human call. |
| **Incident response processes** | Who gets paged, what the escalation path is, how post-incident reviews run — these are team and organisational decisions. PagerDuty MCP can execute the schedule, but someone has to design it. |
| **Penetration testing** | Must be human-led, often third-party. Automated scanning (DAST) is complementary, not a substitute. |
| **Disaster recovery strategy** | Deciding RPO/RTO targets, failover regions, and what "recovered" means is a business decision. MCP servers can execute a DR runbook, but the strategy is human. |
| **Access reviews & privilege audits** | AWS MCP can pull IAM policies and Access Analyzer findings, but deciding who should have what access is a governance exercise. |
| **Backup restoration testing** | Technically scriptable via AWS MCP (restore to temp instance, validate, tear down), but designing what "valid" means and scheduling the tests is human work. |
| **Game days / chaos engineering** | AWS Fault Injection Simulator has APIs, but designing the experiments and interpreting results requires engineering judgment. |

### Coverage Summary

| Category | MCP Coverage | Key Servers |
|---|---|---|
| Cloud Infrastructure | **High** | AWS MCP Server |
| CI/CD Pipeline | **High** | GitHub MCP Server |
| Infrastructure as Code | **High** | Terraform MCP Server |
| Secrets Management | **High** | AWS MCP Server (Secrets Manager APIs) |
| Code Quality & Security | **High** | SonarQube MCP, Snyk MCP |
| Observability | **Medium-High** | Datadog MCP, Grafana Cloud Traces MCP |
| Incident Management | **Medium-High** | PagerDuty MCP |
| Compliance & Governance | **None** | Human judgment required |
| DR & Business Continuity | **None** | Human judgment required |

The practical takeaway: if you connect the AWS, GitHub, Terraform, and one observability MCP server, you can automate a substantial portion of the "user must do" items. Everything in the compliance and governance category stays firmly in human hands.

## Pros of This Approach

**Prevents silent corner-cutting.** The biggest risk with AI-assisted development isn't bad code — it's code that looks good but skips non-functional requirements. The REQUIRE/FLAG pattern makes gaps visible instead of hidden.

**Scales with the team.** A junior developer using Claude Code with this template gets the same guardrails as a senior engineer. The standards are embedded in the tooling, not in tribal knowledge.

**Separates concerns cleanly.** By being explicit about what Claude Code can and can't do, you avoid the frustration of expecting it to provision infrastructure or the risk of it trying to.

**Stays out of the way.** The root file is short. The pillar files are loaded on demand. Claude Code isn't spending half your context window reading rules that don't apply to the current task.

**Opinionated but forkable.** Specific technology choices produce better output from Claude Code (less ambiguity = fewer bad guesses), but every opinion is in one place and easy to swap.

## Cons and Limitations

**Doesn't guarantee compliance.** Claude Code follows instructions probabilistically, not deterministically. These rules significantly improve the baseline, but they are not a substitute for code review, automated checks in CI, or security audits.

**May be too opinionated for some teams.** If your stack is .NET and Azure, the AWS/Go lean won't help. You'll need to fork and adapt, which is expected but still effort.

**Context window pressure is real.** Even with the split-file approach, a long conversation with complex code may push the pillar file contents out of effective context. The root file mitigates this by keeping the most critical rules always present, but it's a real constraint.

**The "flag to user" pattern requires a responsive user.** If Claude Code flags a missing secrets manager and the user ignores it, the flag achieved nothing. This template makes gaps visible but can't force action.

**No runtime enforcement.** These are development-time standards. They don't prevent a developer from manually committing code that violates them. CI checks (SAST, linting, test gates) are the enforcement layer, and those still need to be set up externally.

**Maintenance burden.** Standards evolve. OpenTelemetry APIs change, AWS launches new services, new vulnerability classes emerge. This template needs periodic review, same as any living standard.

## Recommended Developer Tools

### RTK (Rust Token Killer)

[RTK](https://github.com/russ-mcp/rtk) is a CLI proxy that filters and compresses tool output before it reaches Claude Code's context window, saving 60–90% of tokens on common dev operations (git, ls, test runners, Docker, kubectl, etc.). It works via a Claude Code hook that transparently rewrites Bash commands — no changes to your project `CLAUDE.md` required.

RTK is a per-developer tool, not a project standard, so it belongs in your global `~/.claude/` config rather than in the project. Install it once and it works across all your projects:

```bash
cargo install rust_token_killer
rtk init -g --auto-patch
```

This sets up the rewrite hook in `~/.claude/settings.json` and a slim `RTK.md` reference in your global `~/.claude/CLAUDE.md`. Every Bash command Claude Code runs will be automatically routed through RTK's filters — no per-project setup needed.

## Contributing

This template was built through iterative conversation about what actually goes wrong in production software. If you've seen failure modes that aren't covered, or if rules are too prescriptive for practical use, open an issue or PR.

## License

Unlicense
