# 20 — CI/CD & Automation

Understanding, writing, and debugging CI/CD pipelines with Claude — GitHub Actions, build optimization, and deployment strategies.

---

## What You'll Learn

- Anatomy of CI/CD pipelines — stages, jobs, steps, artifacts
- Reading and understanding existing pipeline configurations
- Writing GitHub Actions workflows from scratch
- Debugging failing CI builds systematically
- Optimizing pipelines for speed (caching, parallelization, conditional execution)
- Deployment strategies — blue/green, canary, rolling updates
- Integrating Claude into automation workflows

**Prerequisites**: [04 — Architecture & Dependencies](04-architecture-and-dependencies.md) (you should understand the project's build system) and [06 — Task Execution](06-task-execution.md) (you should understand how to plan and execute changes)

---

## Anatomy of a CI/CD Pipeline

```mermaid
flowchart LR
    A["Trigger<br/>push, PR,<br/>schedule"] --> B["Build<br/>Install deps,<br/>compile"]
    B --> C["Test<br/>Unit, integration,<br/>e2e"]
    C --> D["Analyze<br/>Lint, security,<br/>coverage"]
    D --> E["Package<br/>Docker image,<br/>artifact"]
    E --> F["Deploy<br/>Staging, then<br/>production"]

    style A fill:#e3f2fd,stroke:#1565c0
    style B fill:#fff3e0,stroke:#e65100
    style C fill:#fff3e0,stroke:#e65100
    style D fill:#fff3e0,stroke:#e65100
    style E fill:#fff3e0,stroke:#e65100
    style F fill:#e8f5e9,stroke:#2e7d32
```

| Concept | What It Is | Example |
|---------|-----------|---------|
| **Pipeline** | The full workflow from trigger to deploy | A GitHub Actions workflow file |
| **Stage** | A logical group of related jobs | "Test", "Build", "Deploy" |
| **Job** | A unit of work that runs on one machine | "Run unit tests", "Build Docker image" |
| **Step** | A single command or action within a job | `npm test`, `docker build` |
| **Artifact** | A file produced by one job, consumed by another | Test report, Docker image, compiled binary |
| **Environment** | A deployment target with its own variables/secrets | staging, production |

---

## Understanding Existing Pipelines

```
Read our CI/CD configuration and explain:

1. What triggers the pipeline? (push, PR, schedule, manual)
2. What jobs run, and in what order?
3. Which jobs depend on other jobs?
4. What caching is configured?
5. How long does each job typically take?
6. Where are secrets and environment variables defined?
7. What's the deployment process?

Draw a diagram showing the job dependency graph.
```

### Reading GitHub Actions Config

```
Read .github/workflows/ and explain each workflow:

For each workflow file:
- When does it trigger?
- What jobs does it contain?
- What does each step do?
- Are there any matrix strategies?
- What actions (third-party) does it use?
- Are there any potential security concerns?
  (e.g., using pull_request_target, untrusted input
  in run commands)
```

---

## Writing GitHub Actions Workflows

### Basic Workflow Structure

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm test

  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
```

### Common Patterns

**Matrix builds — test across multiple versions:**
```yaml
jobs:
  test:
    strategy:
      matrix:
        node-version: [18, 20, 22]
        os: [ubuntu-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm test
```

**Conditional execution — skip expensive jobs for docs-only changes:**
```yaml
jobs:
  check-changes:
    runs-on: ubuntu-latest
    outputs:
      should-test: ${{ steps.filter.outputs.src }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            src:
              - 'src/**'
              - 'tests/**'
              - 'package.json'

  test:
    needs: check-changes
    if: needs.check-changes.outputs.should-test == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test
```

**Service containers — database for integration tests:**
```yaml
jobs:
  integration-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test_db
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run test:integration
        env:
          DATABASE_URL: postgres://postgres:test@localhost:5432/test_db
```

### Writing a Workflow from Scratch

```
Help me write a GitHub Actions workflow for our project:

Build system: [npm/yarn/pnpm/make/gradle/etc.]
Language: [Node/Python/Go/etc.]
Tests: [test command and what framework]
Lint: [lint command]
Other checks: [type checking, security scanning, etc.]
Deploy target: [Vercel/AWS/GCP/Docker registry/etc.]

Requirements:
- Run on PRs and pushes to main
- Cache dependencies between runs
- Run tests and lint in parallel
- Only deploy on push to main (not PRs)
- Fail fast if any check fails
```

---

## Debugging Failing CI Builds

### CI Failure Diagnosis

```mermaid
flowchart TD
    A[CI Build<br/>Failed] --> B{Same failure<br/>locally?}
    B -->|Yes| C["Debug locally<br/>(faster iteration)"]
    B -->|No| D{Environment<br/>difference?}
    D -->|Node/Python<br/>version| E["Pin version in<br/>workflow and<br/>local setup"]
    D -->|OS difference| F["Check for<br/>path/case/line-ending<br/>issues"]
    D -->|Missing env var<br/>or secret| G["Check secrets<br/>config and<br/>environment"]
    D -->|Timing/Race| H["Flaky test —<br/>add retry or<br/>fix the race"]
    A --> I{Error type?}
    I -->|OOM killed| J["Increase runner<br/>memory or split<br/>the job"]
    I -->|Timeout| K["Optimize or<br/>increase timeout"]
    I -->|Permission denied| L["Check file<br/>permissions and<br/>checkout settings"]
    I -->|Network error| M["Retry step or<br/>check firewall<br/>rules"]

    style C fill:#e8f5e9,stroke:#2e7d32
    style E fill:#fff3e0,stroke:#e65100
    style H fill:#fff3e0,stroke:#e65100
```

### Common CI Failures

```
Our CI build is failing. Here's the error log:

[paste the relevant portion of the CI log]

Help me diagnose:
1. What's the actual error? (not just the symptom)
2. Is this a code issue or an environment/config issue?
3. Can I reproduce this locally? How?
4. What's the fix?
```

| Failure Type | Symptoms | Common Causes |
|-------------|----------|---------------|
| **Dependency install** | `npm ci` fails, lock file mismatch | Lock file not committed, version conflicts |
| **Flaky tests** | Pass sometimes, fail sometimes | Race conditions, time-dependent, external service dependency |
| **OOM killed** | Process killed, exit code 137 | Test memory leak, too many parallel processes |
| **Timeout** | Job exceeds time limit | Stuck process, missing test teardown, slow external call |
| **Permission denied** | Can't read/write files | File permissions, missing secrets, wrong runner |
| **Version mismatch** | Works locally, fails in CI | Different Node/Python/Go version, different OS |

### Reproducing CI Locally

```
Help me reproduce this CI failure locally:

1. What exact versions is CI using? (check the workflow)
2. What environment variables are set?
3. What's the exact command sequence?
4. Are there service containers I need running?

Give me the commands to set up an identical local
environment and reproduce the failure.
```

---

## Pipeline Optimization

### Caching

Cache anything that's slow to compute and changes infrequently:

```yaml
# Node.js — cache node_modules via npm cache
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: 'npm'

# Custom caching — e.g., build output
- uses: actions/cache@v4
  with:
    path: |
      .next/cache
      dist/
    key: build-${{ runner.os }}-${{ hashFiles('src/**') }}
    restore-keys: |
      build-${{ runner.os }}-
```

### Parallelization

Run independent jobs concurrently:

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run lint

  test-unit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test -- --shard=1/2

  test-unit-2:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test -- --shard=2/2

  test-integration:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run test:integration

  # Only runs if all checks pass
  deploy:
    needs: [lint, test-unit, test-unit-2, test-integration]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploy!"
```

### Optimization Checklist

```
Analyze our CI pipeline for optimization opportunities:

1. Which jobs take the longest? Can they be parallelized?
2. Are we caching effectively? (dependencies, build output)
3. Are we running unnecessary checks on non-code changes?
4. Can we use test sharding to split test suites?
5. Are we installing dev dependencies when only production
   ones are needed?
6. Do we have duplicate setup steps across jobs?
7. Can we use a lighter runner image?

Current pipeline time: [X minutes]
Target: [Y minutes]
```

---

## Deployment Strategies

| Strategy | How It Works | Risk Level | Rollback Speed |
|----------|-------------|------------|----------------|
| **Rolling** | Replace instances gradually | Medium | Medium (re-deploy old version) |
| **Blue/Green** | Run two identical environments, switch traffic | Low | Instant (switch back) |
| **Canary** | Route small % of traffic to new version | Low | Fast (route 0% to canary) |
| **Recreate** | Stop old, start new | High | Slow (downtime during deploy) |

### Blue/Green Deployment

```mermaid
flowchart LR
    LB["Load<br/>Balancer"] --> Blue["Blue (v1)<br/>Current"]
    LB -.->|"switch"| Green["Green (v2)<br/>New"]

    style Blue fill:#e3f2fd,stroke:#1565c0
    style Green fill:#e8f5e9,stroke:#2e7d32
```

1. Blue is live, serving all traffic
2. Deploy v2 to Green
3. Run smoke tests against Green
4. Switch load balancer to Green
5. Blue becomes the standby (instant rollback = switch back)

### Canary Deployment

```mermaid
flowchart LR
    LB["Load<br/>Balancer"] -->|"95%"| Stable["Stable (v1)<br/>Current"]
    LB -->|"5%"| Canary["Canary (v2)<br/>New"]

    style Stable fill:#e3f2fd,stroke:#1565c0
    style Canary fill:#fff3e0,stroke:#e65100
```

1. Deploy v2 to a small pool (canary)
2. Route 5% of traffic to canary
3. Monitor error rates, latency, business metrics
4. Gradually increase (5% → 25% → 50% → 100%)
5. If metrics degrade at any point, route 0% to canary

```
Help me design a deployment strategy for our service:

- Current setup: [describe infrastructure]
- Deployment frequency: [daily/weekly/etc.]
- Downtime tolerance: [zero/minutes/flexible]
- Rollback requirements: [instant/minutes/acceptable delay]
- Traffic volume: [requests per second]
- Database migrations involved? [yes/no]
```

---

## Integrating Claude into Automation

### Automated PR Review

```yaml
# .github/workflows/claude-review.yml
name: Claude PR Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Get diff
        run: git diff origin/main...HEAD > diff.txt
      - name: Claude review
        # Use Claude API or Claude Code CLI to review
        # the diff and post comments
```

### Other Automation Ideas

- **Documentation generation**: Regenerate API docs on each merge to main
- **Changelog generation**: Summarize changes since last release
- **Migration safety check**: Analyze database migration files for risky operations
- **Dependency update review**: Summarize what changed in dependency updates

---

## Sample Conversation

> **You**: Our CI pipeline takes 25 minutes. It needs to be under 10. Here's the workflow file. Help me optimize it.
>
> **Claude**: *reads the workflow file*
>
> Here's the breakdown of your current pipeline:
>
> | Job | Duration | Depends On |
> |-----|----------|-----------|
> | Install deps | 3 min | — |
> | Lint | 2 min | Install |
> | Type check | 2 min | Install |
> | Unit tests | 8 min | Install |
> | Integration tests | 6 min | Install |
> | Build | 3 min | All tests |
> | Deploy | 1 min | Build |
> | **Total (serial)** | **25 min** | |
>
> **Problem**: Everything runs sequentially, and each job re-installs dependencies.
>
> **Optimized plan:**
>
> 1. **Cache dependencies** — `npm ci` with cache drops install from 3 min to 30 sec
> 2. **Parallelize independent jobs** — lint, type check, unit tests, and integration tests run simultaneously
> 3. **Shard unit tests** — split across 2 runners (8 min → 4 min)
> 4. **Skip unchanged** — use path filters to skip tests when only docs change
>
> **New timeline:**
> ```
> ├─ Lint (2 min)          ─┐
> ├─ Type check (2 min)     ├─ All parallel
> ├─ Unit tests 1/2 (4 min) │
> ├─ Unit tests 2/2 (4 min) │
> ├─ Integration (6 min)   ─┘
> └─ Build + Deploy (4 min)  ← after all pass
>
> Total: ~10 min (longest parallel path: 6 min + 4 min)
> ```
>
> With caching, the longest path becomes about 7 minutes. Want me to rewrite the workflow file?
>
> **You**: Yes, do it.
>
> **Claude**: *rewrites the workflow with caching, parallel jobs, test sharding, and path filters — runs the workflow syntax checker*

---

## Key Takeaways

1. Understand your pipeline before optimizing it — read the config and map the dependency graph
2. Parallelize independent jobs — lint, test, and type check don't need to wait for each other
3. Cache aggressively — dependencies, build output, anything slow and infrequently changing
4. Skip work that isn't needed — use path filters to avoid running tests on docs-only changes
5. Debug CI failures systematically — reproduce locally first, then check for environment differences
6. Choose deployment strategy based on your rollback needs — blue/green for instant rollback, canary for gradual confidence
7. Pin versions everywhere — Node, Python, actions, runners — to prevent surprise breakage

---

**Next**: [21 — Performance Optimization](21-performance-optimization.md) — Proactive performance optimization with profiling, caching, and load testing.
