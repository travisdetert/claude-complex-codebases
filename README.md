# Navigating Complex Codebases with Claude Code

A practical, modular guide for using [Claude Code](https://docs.anthropic.com/en/docs/claude-code) to understand, navigate, and work effectively in large or unfamiliar codebases.

Whether you've never used Claude Code or you're looking for advanced workflows, this guide series walks you through everything — from installation to ongoing maintenance practices.

---

## Roadmap

```mermaid
flowchart LR
    A["01<br/>Getting Started"] --> B["02<br/>Setup &<br/>Configuration"]
    B --> C["03<br/>Codebase<br/>Orientation"]
    C --> D["04<br/>Architecture &<br/>Dependencies"]
    D --> E["05<br/>Codebase<br/>Archaeology"]
    E --> F["06<br/>Task<br/>Execution"]
    F --> G["07<br/>Diagrams &<br/>Docs"]
    G --> H["08<br/>Ongoing<br/>Practices"]
    H --> I["09<br/>Anti-<br/>Patterns"]
    I --> J["10<br/>Migration<br/>Planning"]
    J --> K["11<br/>Local Env<br/>Setup"]
    K --> L["12<br/>Debugging &<br/>Troubleshooting"]
    L --> M["13<br/>Code<br/>Review"]
    M --> N["14<br/>Testing<br/>Strategies"]
    N --> O["15<br/>Security<br/>Analysis"]
    O --> P["16<br/>Legacy<br/>Code"]
    P --> Q["17<br/>Collaboration<br/>& Teams"]
    Q --> R["18<br/>API Design<br/>& Evolution"]
    R --> S["19<br/>Data Modeling<br/>& DB Design"]
    S --> T["20<br/>CI/CD &<br/>Automation"]
    T --> U["21<br/>Performance<br/>Optimization"]
    U --> V["22<br/>Incident<br/>Response"]
    V --> W["23<br/>Tech Debt<br/>Management"]
    W --> X["24<br/>Accessibility<br/>Auditing"]

    style A fill:#e8f5e9,stroke:#2e7d32
    style B fill:#e8f5e9,stroke:#2e7d32
    style C fill:#e3f2fd,stroke:#1565c0
    style D fill:#e3f2fd,stroke:#1565c0
    style E fill:#e3f2fd,stroke:#1565c0
    style F fill:#fff3e0,stroke:#e65100
    style G fill:#fff3e0,stroke:#e65100
    style H fill:#f3e5f5,stroke:#6a1b9a
    style I fill:#f3e5f5,stroke:#6a1b9a
    style J fill:#fff3e0,stroke:#e65100
    style K fill:#fff3e0,stroke:#e65100
    style L fill:#fff3e0,stroke:#e65100
    style M fill:#fff3e0,stroke:#e65100
    style N fill:#fff3e0,stroke:#e65100
    style O fill:#fff3e0,stroke:#e65100
    style P fill:#fff3e0,stroke:#e65100
    style Q fill:#f3e5f5,stroke:#6a1b9a
    style R fill:#fff3e0,stroke:#e65100
    style S fill:#fff3e0,stroke:#e65100
    style T fill:#fff3e0,stroke:#e65100
    style U fill:#fff3e0,stroke:#e65100
    style V fill:#fff3e0,stroke:#e65100
    style W fill:#f3e5f5,stroke:#6a1b9a
    style X fill:#fff3e0,stroke:#e65100
```

**Green** = Setup | **Blue** = Understanding | **Orange** = Doing | **Purple** = Sustaining

---

## Guides

| # | Guide | Description |
|---|-------|-------------|
| 01 | [Getting Started](guides/01-getting-started.md) | Installation, first launch, key concepts, your first 5-minute exploration |
| 02 | [Setup & Configuration](guides/02-setup-and-configuration.md) | CLAUDE.md, permissions, the `.claude/` directory, MCP servers, team workflows |
| 03 | [Codebase Orientation](guides/03-codebase-orientation.md) | Big-picture analysis, entry points, request tracing, strategies by project type |
| 04 | [Architecture & Dependencies](guides/04-architecture-and-dependencies.md) | Build systems, databases, dependency analysis, architecture diagrams |
| 05 | [Codebase Archaeology](guides/05-codebase-archaeology.md) | Git history, pain points, cross-cutting concerns, tribal knowledge |
| 06 | [Task Execution](guides/06-task-execution.md) | Plan mode, tests-first, blast radius, making changes, working with PRs |
| 07 | [Diagrams & Documentation](guides/07-diagrams-and-documentation.md) | Mermaid diagram catalog, generating docs, onboarding materials |
| 08 | [Ongoing Practices](guides/08-ongoing-practices.md) | Session management, CLAUDE.md evolution, verification habits, knowledge building |
| 09 | [Anti-Patterns](guides/09-anti-patterns.md) | Common mistakes with examples, consequences, and fixes |
| 10 | [Migration Planning](guides/10-migration-planning.md) | Assessing scope, incremental strategies, database migrations, rollback planning |
| 11 | [Local Environment Setup](guides/11-local-environment-setup.md) | From `git clone` to running — Docker, databases, services, troubleshooting setup failures |
| 12 | [Debugging & Troubleshooting](guides/12-debugging-and-troubleshooting.md) | Systematic debugging — logs, stack traces, performance, flaky tests, the debugging decision tree |
| 13 | [Code Review](guides/13-code-review.md) | Reviewing PRs — understanding diffs, tracing impact, spotting issues, writing constructive comments |
| 14 | [Testing Strategies](guides/14-testing-strategies.md) | Writing effective tests — unit/integration/e2e, edge cases, mocking, hard-to-test code |
| 15 | [Security Analysis](guides/15-security-analysis.md) | Auditing for vulnerabilities — OWASP Top 10, auth flows, injection risks, dependency CVEs |
| 16 | [Working with Legacy Code](guides/16-working-with-legacy-code.md) | Safely modifying untested code — characterization tests, strangler fig, incremental improvement |
| 17 | [Collaboration & Team Workflows](guides/17-collaboration-and-team-workflows.md) | Team practices — shared CLAUDE.md, onboarding, pair programming, knowledge capture |
| 18 | [API Design & Evolution](guides/18-api-design-and-evolution.md) | Designing APIs — REST/GraphQL conventions, versioning, backward compatibility, deprecation strategies |
| 19 | [Data Modeling & Database Design](guides/19-data-modeling-and-database-design.md) | Schema design — normalization, indexing strategies, query optimization, ER modeling |
| 20 | [CI/CD & Automation](guides/20-ci-cd-and-automation.md) | Pipelines — GitHub Actions, debugging builds, deployment strategies, integrating Claude into automation |
| 21 | [Performance Optimization](guides/21-performance-optimization.md) | Proactive optimization — profiling, caching strategies, load testing, database tuning |
| 22 | [Incident Response](guides/22-incident-response.md) | Production incidents — triage, rollback decisions, postmortems, on-call with Claude |
| 23 | [Technical Debt Management](guides/23-technical-debt-management.md) | Systematic debt management — identifying, measuring, prioritizing, and paying down tech debt |
| 24 | [Accessibility Auditing](guides/24-accessibility-auditing.md) | WCAG compliance — semantic HTML, keyboard navigation, screen readers, color contrast |

---

## Where to Start

**Brand new to Claude Code?**
Start with [01 — Getting Started](guides/01-getting-started.md) and follow the guides in order.

**Already using Claude Code, new to a codebase?**
Skim [02 — Setup & Configuration](guides/02-setup-and-configuration.md), then start at [03 — Codebase Orientation](guides/03-codebase-orientation.md).

**Working on a specific task right now?**
Jump to [06 — Task Execution](guides/06-task-execution.md), but consider skimming [03](guides/03-codebase-orientation.md) first if the codebase is unfamiliar.

**Want to improve your workflow?**
Read [08 — Ongoing Practices](guides/08-ongoing-practices.md) and [09 — Anti-Patterns](guides/09-anti-patterns.md).

**Struggling to get a project running locally?**
Jump to [11 — Local Environment Setup](guides/11-local-environment-setup.md) for help with Docker, databases, and service dependencies.

**Debugging a tricky issue?**
Jump to [12 — Debugging & Troubleshooting](guides/12-debugging-and-troubleshooting.md) for a systematic approach to finding root causes.

**Reviewing a PR or writing tests?**
Jump to [13 — Code Review](guides/13-code-review.md) or [14 — Testing Strategies](guides/14-testing-strategies.md) for systematic approaches to review and test coverage.

**Concerned about security?**
Jump to [15 — Security Analysis](guides/15-security-analysis.md) for vulnerability auditing with the OWASP Top 10 checklist.

**Working with old, untested code?**
Jump to [16 — Working with Legacy Code](guides/16-working-with-legacy-code.md) for safe modification patterns.

**Setting up team workflows?**
Jump to [17 — Collaboration & Team Workflows](guides/17-collaboration-and-team-workflows.md) for shared CLAUDE.md practices and onboarding.

**Planning a large migration?**
Jump to [10 — Migration Planning](guides/10-migration-planning.md) — but read [03](guides/03-codebase-orientation.md) and [06](guides/06-task-execution.md) first if the codebase is unfamiliar.

**Designing an API?**
Jump to [18 — API Design & Evolution](guides/18-api-design-and-evolution.md) for REST/GraphQL conventions, versioning, and backward compatibility.

**Designing a database schema?**
Jump to [19 — Data Modeling & Database Design](guides/19-data-modeling-and-database-design.md) for normalization, indexing, and query optimization.

**Setting up or fixing CI/CD pipelines?**
Jump to [20 — CI/CD & Automation](guides/20-ci-cd-and-automation.md) for GitHub Actions, debugging builds, and deployment strategies.

**Optimizing for performance?**
Jump to [21 — Performance Optimization](guides/21-performance-optimization.md) for profiling, caching, and load testing.

**Dealing with a production incident?**
Jump to [22 — Incident Response](guides/22-incident-response.md) for triage, rollback decisions, and blameless postmortems.

**Tackling technical debt?**
Jump to [23 — Technical Debt Management](guides/23-technical-debt-management.md) for systematic identification, prioritization, and repayment.

**Making your app accessible?**
Jump to [24 — Accessibility Auditing](guides/24-accessibility-auditing.md) for WCAG compliance, keyboard navigation, and screen reader testing.

---

## Further Reading

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [CLAUDE.md Best Practices](https://docs.anthropic.com/en/docs/claude-code/memory)
