# Claude Code Standards Template

Drop-in engineering standards for Claude Code. Copy two things into your project — a `CLAUDE.md` file and a `docs/standards/` folder — and Claude Code starts enforcing security, testing, tenant isolation, structured logging, and observability by default.

## Quick Start

```bash
# Copy into your project
cp CLAUDE.md /path/to/your/project/
cp -r docs/standards/ /path/to/your/project/docs/standards/

# Tailor it (run this prompt in Claude Code):
# "Review CLAUDE.md and docs/standards/. Remove anything irrelevant
#  to [describe your project]. Update anything outdated."

# Start building. Standards are enforced automatically.
```

## FAQ

**What does this actually do?**
It gives Claude Code a persistent instruction set that prevents the most common production failures: hardcoded secrets, missing tenant isolation, swallowed exceptions, skipped tests, unstructured logging, and unscoped data access. It doesn't just tell Claude what to build — it tells Claude what not to skip.

**How big is the context overhead?**
The root `CLAUDE.md` is ~75 lines. That's always loaded. The 12 detailed pillar files in `docs/standards/` are only loaded on demand when Claude works in a relevant domain (e.g., `database.md` loads when you're writing migrations). Most conversations only load 1-2 pillar files.

**Do I need all of it?**
No. A CLI tool doesn't need tenant isolation. A static site doesn't need correlation IDs. Run the tailoring prompt in Quick Start to strip what doesn't apply. The template is deliberately opinionated — fork it and cut what you don't need.

**Does it guarantee compliance?**
No. It provides technical controls that compliance frameworks like GDPR, PCI-DSS, and APRA CPS 234 require, but compliance is an organisational and legal exercise. This is a guardrail, not a certification.

**Does Claude always follow the rules?**
No. Claude follows instructions probabilistically. These rules significantly improve the baseline, but in long sessions, context pressure can push pillar files out of effective memory. You still need code review, CI checks, and security audits.

**What technology stack does it assume?**
Go primary, Python 3 (strict mypy) secondary, TypeScript strict for frontend. AWS serverless. OpenTelemetry. Structured JSON logging. These are opinions — change them in `CLAUDE.md` to match your stack.

**Is there an automated setup for the developer tools?**
Yes. [claude-tools](https://github.com/leighstillard/claude-tools) is a companion repo that installs RTK, LSP servers, and Claude Code plugins in one command. It's kept separate so this repo stays focused on engineering standards only.

**What's the difference between this and just writing good prompts?**
Prompts are per-conversation. This is persistent — it loads automatically in every session. You don't have to remember to ask for tenant isolation or structured logging. The standards are always active.

---

## What's Inside

### The Root File (~75 lines, always loaded)

`CLAUDE.md` contains the non-negotiable rules: no hardcoded secrets, parameterised queries only, tenant-scoped data access, structured logging, test coverage requirements, and MCP server safety. These are the rules that prevent the worst outcomes and are always in context.

### The Pillar Files (loaded on demand)

```
docs/standards/
  security.md              # Input validation, auth, headers, crypto
  data-privacy.md          # Tenant isolation, PII handling, retention
  secrets.md               # Config modules, env vars, pre-commit hooks
  testing.md               # TDD workflow, coverage categories, gap analysis
  error-handling.md        # Structured logging, correlation IDs, log levels
  observability.md         # OpenTelemetry, golden signals, health checks
  api-design.md            # Versioning, pagination, error schemas
  database.md              # Migrations, connection pooling, tenant scoping
  code-quality.md          # Project structure, naming, dependencies
  scaffolding.md           # Dockerfiles, CI configs, IaC, runbooks
  mcp-safety.md            # MCP server guardrails, cost traps, risk warnings
  workflow-discipline.md   # Assumption surfacing, simplicity, surgical changes
```

Each pillar uses a `DO / DO NOT / REQUIRE` structure. DO rules describe good practice. DO NOT rules block common shortcuts. REQUIRE rules flag missing external dependencies (no secrets manager configured, no auth provider) so Claude raises them to you instead of silently working around them.

### Three-Tier Responsibility Model

| Tier | Who Does It | Examples |
|---|---|---|
| **Claude Code does it** | Enforced in code | Input validation, parameterised queries, tenant-scoped access, structured logging, test coverage |
| **Claude Code scaffolds it** | Claude generates, you deploy | Dockerfiles, CI/CD configs, IaC templates, runbooks |
| **You do it** | Human judgment required | Secrets infrastructure, compliance decisions, SLO targets, incident response, penetration testing |

## PR Workflow & Review Checklist

The template includes a tiered review checklist using Claude Code plugins.

### Required Plugins

| Plugin | Provides | Install |
|---|---|---|
| **superpowers-extended-cc** | `/simplify`, code review, planning, TDD | `/plugin marketplace add pcvelz/superpowers` then `/plugin install superpowers-extended-cc@superpowers-extended-cc-marketplace` |
| **code-review** | `/code-review` — PR-level review | `/plugin marketplace add anthropics/code-review` then `/plugin install code-review@code-review-marketplace` |
| **security-review** | `/security-review` — security review | Install via your plugin manager |

### Review Tiers

**Major milestones** (new features, architectural changes, multi-file work):
1. `/simplify` — code cleanup
2. `/security-review` — security check
3. `superpowers:requesting-code-review` — pre-PR self-check
4. `/code-review` — full PR review

**Minor fixes** (bug fixes, small tweaks, single-file changes):
1. `/simplify` — code cleanup
2. `/security-review` — security check
3. `/code-review` — PR review

## Recommended Developer Tools

### Automated Setup

Install everything below in one step with [claude-tools](https://github.com/leighstillard/claude-tools). It's a companion repo kept separate so this one stays pure standards.

### RTK (Rust Token Killer)

[RTK](https://github.com/rtk-ai/rtk) compresses CLI output before it reaches Claude's context window — 60-90% token savings on git, ls, test runners, Docker, kubectl, etc. Works via a transparent hook; no project config needed.

```bash
cargo install rust_token_killer
rtk init -g --auto-patch
```

### LSP (Language Server Protocol)

[LSP in Claude Code](https://karanbansal.in/blog/claude-code-lsp/) gives it semantic code understanding — go-to-definition, find-references, and type lookups in ~50ms instead of 30-60 seconds of grep-based searching. When Claude modifies a function signature, LSP flags every broken call site so Claude fixes them in one turn.

Add `"ENABLE_LSP_TOOL": "1"` to `~/.claude/settings.json` and install language servers:

| Language | Server |
|---|---|
| Python | `pyright` |
| TypeScript / JavaScript | `typescript-language-server` |
| Go | `gopls` |

Ensure the corresponding Claude Code plugins are both installed **and enabled**. Restart Claude Code after setup.

## MCP Server Safety

> MCP servers that can write to cloud infrastructure are giving an AI agent access to real resources and real money. Read `docs/standards/mcp-safety.md` before connecting any MCP server with write access.

### Recommended Servers

| Concern | Server | Risk | Notes |
|---|---|---|---|
| AWS Infrastructure | [AWS MCP](https://github.com/awslabs/mcp) | Medium | **Read-only credentials only.** Propose changes as CloudFormation templates. |
| CI/CD & Source Control | [GitHub MCP](https://github.com/github/github-mcp-server) | Medium | Can merge PRs, delete branches, modify workflows. Use fine-grained tokens. |
| Infrastructure as Code | [Terraform MCP](https://github.com/hashicorp/terraform-mcp-server) | High | `apply` modifies real infrastructure. `destroy` is irreversible. |
| Observability | [Datadog MCP](https://www.datadoghq.com/blog/datadog-remote-mcp-server/), [Grafana Traces MCP](https://grafana.com/docs/grafana-cloud/send-data/traces/mcp-server/) | Low | Primarily read-only. |
| Code Quality | [SonarQube MCP](https://github.com/SonarSource/sonarqube-mcp-server), [Snyk MCP](https://snyk.io/) | Low | Primarily read-only. |
| Incidents | [PagerDuty MCP](https://github.com/PagerDuty/pagerduty-mcp-server) | Medium | Can page real humans. Use read-only mode by default. |

### What MCP Can't Do

Data classification policy, regulatory mapping, SLO/SLA targets, penetration testing, disaster recovery strategy, access reviews, and incident response design all require human judgment. See `docs/standards/mcp-safety.md` for the full breakdown.

## Contributing

If you've seen failure modes that aren't covered, or if rules are too prescriptive for practical use, open an issue or PR.

## License

Unlicense
