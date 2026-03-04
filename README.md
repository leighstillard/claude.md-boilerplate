# Claude Code Standards Template

An opinionated `claude.md` template that steers Claude Code toward production-grade software engineering practices. Built for teams that care about security, data privacy, observability, and testability — not just "does it run."

## The Problem

Claude Code is remarkably capable at generating functional software. But "functional" and "production-ready" are different standards. Without guidance, AI-assisted development tends to produce code that works but cuts corners on the things that matter when software runs in the real world: tenant isolation, structured logging, secret handling, negative-path testing, and graceful failure.

A `claude.md` file is Claude Code's persistent instruction set — it's loaded automatically and shapes every interaction. This repo provides a template that uses that mechanism to enforce engineering discipline by default.

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
```

## Technology Opinions

The template makes deliberate technology choices to reduce ambiguity:

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

During design, we evaluated which "user must do" items could potentially be automated via MCP (Model Context Protocol) servers. The findings:

- **High MCP coverage**: Secrets infrastructure (AWS MCP), CI/CD pipeline setup (GitHub MCP), infrastructure provisioning (AWS/Terraform MCP), network and DNS configuration
- **Medium coverage**: Observability platform setup (Grafana/Datadog APIs), dashboard and alerting configuration, container registry management
- **Low coverage**: Compliance decisions, incident response processes, SLO definition, penetration testing, access reviews

This analysis informed the three-tier model. Items with high MCP coverage sit in the "scaffold and flag" tier because they're automatable today with the right MCP servers connected — but they still require the user to provide credentials and approve actions.

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

## How to Use

1. Copy `claude.md` and the `docs/standards/` directory into your project root.
2. Review the technology opinions and adjust to your stack.
3. Remove any pillar files that don't apply (e.g., `data-privacy.md` if you're building a single-tenant tool).
4. Start a Claude Code session. The root `claude.md` loads automatically.
5. When Claude Code works in a specific domain, it will reference the relevant pillar file for detailed guidance.

## Contributing

This template was built through iterative conversation about what actually goes wrong in production software. If you've seen failure modes that aren't covered, or if rules are too prescriptive for practical use, open an issue or PR.

## License

Unlicense