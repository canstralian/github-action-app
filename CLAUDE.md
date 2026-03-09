# CLAUDE.md — Development Workflow Guide for Trading Bot Swarm

This guide standardizes how **GitHub Copilot** and **Codex** are configured and used across the Trading Bot Swarm ecosystem so we ship reliable, secure, and test-verified changes.

---

## 1) Purpose and Scope

### Purpose
- Align human contributors and AI assistants on one development standard.
- Use Copilot as a **pair programmer** with strict behavioral boundaries.
- Use Codex as a **workflow executor** that always validates changes with quality gates.
- Reduce regressions through repeatable test/lint/review automation.

### Scope
This guide covers:
- Agent behavior and operational rules.
- Skills and tool/connector usage patterns.
- Coding, testing, linting, security, observability, and CI/CD defaults.
- Versioning, release, dependency hygiene, and contributor workflows.

---

## 2) Working Model: Agents, Skills, and Connectors/Tools

### Agent Roles
- **Human developer**: Defines intent, architecture, and acceptance criteria.
- **Copilot**: Suggests code inline, constrained by repository rules.
- **Codex**: Executes deterministic change workflows (edit, test, commit, PR).

### Skills Available in This Environment
- **`skill-creator`**: Used for creating/updating skills with reusable, specialized workflows.
- **`skill-installer`**: Used to list/install curated skills or install from GitHub paths.

### Connector/Tool Capabilities Available
- Shell execution for local checks and automation.
- MCP resource listing/reading for structured context retrieval.
- Browser automation/screenshot tooling for UI validation when relevant.
- PR metadata creation tooling for standardized pull request output.

### Recommended Orchestration Sequence
1. **Read context first** (repo docs, active instructions, existing standards).
2. **Select minimal skill set** needed for task completion.
3. **Implement smallest safe change**.
4. **Run quality gates** (lint + tests; docs-only changes may skip runtime tests).
5. **Commit with Conventional Commit message**.
6. **Open PR with clear risk/validation notes**.

---

## 3) Copilot as a Pair Programmer (Strict Rules)

Copilot should be treated as an assistant that proposes code, not an authority.

### Mandatory Behavior Rules
- Never bypass tests/linters for code changes.
- Never introduce secrets, insecure defaults, or broad permissions.
- Never refactor unrelated modules without explicit request.
- Never change release/version files outside intended scope.
- Prefer small, reviewable patches over large rewrites.
- Ask for deterministic outputs (typed interfaces, explicit error paths, stable tests).

### Example Prompting Rules for Developers
- “Generate code that satisfies existing lint rules and test conventions.”
- “Preserve current public interfaces unless the task explicitly changes them.”
- “Add/adjust tests for all behavior changes.”
- “For docs-only edits, do not alter runtime logic.”

---

## 4) Configuration Overview (Engineering Standards)

### Testing
- Run unit tests on every PR.
- Run integration tests for strategy execution, exchange adapters, and order lifecycle logic.
- Enforce minimum coverage threshold (example: 80%+ on changed modules).

### Linting and Formatting
- Enforce lint checks in CI (ESLint/Ruff/Flake8 equivalent stack).
- Enforce formatting (Prettier/Black/go fmt equivalent stack).
- Block merges on lint failures.

### Code Style
- Small functions, explicit naming, clear boundaries.
- Prefer pure functions for signal generation and risk calculations.
- Centralize constants for limits, slippage, fee assumptions, and retry budgets.

### Async and Concurrency Patterns
- Use cancellation-aware async flows.
- Enforce timeouts and retry policies for external I/O.
- Ensure idempotency for order placement/reconciliation routines.

### Security Defaults
- Principle of least privilege for tokens and CI permissions.
- No plaintext secrets; use environment secrets manager.
- Pin third-party actions/dependencies where possible.
- Validate all exchange/webhook payloads and signatures.

### Logging and Observability
- Structured logs (JSON) with correlation IDs.
- Metrics for order latency, error rate, rejection rate, and PnL drift.
- Alerting thresholds for failed executions, stale market data, and risk breaches.

### CI/CD Integration
- Branch protection: require status checks + review approvals.
- Quality gates before merge.
- Optional deployment gates for production strategy rollout.

### Version Control
- Conventional Commits required.
- Short-lived feature branches.
- Squash merge for clean history unless release process requires merge commits.

---

## 5) Custom Instructions for Codex and Copilot

Below is a conceptual YAML format to standardize assistant behavior.

```yaml
ai_assistant_policy:
  scope: trading-bot-swarm
  priorities:
    - safety
    - correctness
    - testability
    - maintainability

copilot:
  role: pair_programmer
  rules:
    - "Generate minimal diffs that satisfy task scope."
    - "Do not modify unrelated files."
    - "For code changes, include/update tests."
    - "For docs-only changes, skip runtime code edits."
    - "Respect existing lint, format, and typing standards."
    - "Never include secrets, tokens, or credentials."

codex:
  role: implementation_executor
  rules:
    - "Read repository instructions before editing."
    - "Run lint and tests on code changes before commit."
    - "Allow docs-only changes to bypass runtime test suites."
    - "Use conventional commit messages."
    - "Summarize validation commands in PR body."

quality_gate:
  run_on:
    - pull_request
    - push_to_main
  checks:
    - lint
    - test
    - dependency_audit
    - secret_scan
  docs_only_behavior:
    lint_docs: true
    run_runtime_tests: false
```

---

## 6) GitHub Workflow Example: Lint + Test Automation

### Trigger Conditions
- On `pull_request` to `main`.
- On `push` to `main`.
- Optional manual trigger (`workflow_dispatch`) for diagnostics.

### Quality Gate Job Steps
1. Checkout repository.
2. Setup runtime/toolchain and dependency cache.
3. Install dependencies.
4. Run lint.
5. Run tests.
6. Upload reports/artifacts.

```yaml
name: quality-gate

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  quality-gate:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup runtime
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Test
        run: npm test -- --ci

      - name: Upload test artifacts
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: test-results/
```

---

## 7) Best-Practice Workflow: Semantic Release and Version Tagging

Use automated semantic versioning based on Conventional Commits.

```yaml
name: release

on:
  push:
    branches: [main]

jobs:
  semantic-release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      issues: write
      pull-requests: write
      id-token: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npx semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

Versioning conventions:
- `feat:` → minor bump.
- `fix:` → patch bump.
- `BREAKING CHANGE:` → major bump.

---

## 8) Best-Practice Workflow: Security and Dependency Scanning

```yaml
name: security-and-deps

on:
  pull_request:
  push:
    branches: [main]
  schedule:
    - cron: '0 3 * * *'

jobs:
  scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v4

      - name: Dependency audit
        run: npm audit --audit-level=high

      - name: CodeQL init
        uses: github/codeql-action/init@v3
        with:
          languages: javascript

      - name: CodeQL analyze
        uses: github/codeql-action/analyze@v3
```

Security baseline:
- Rotate keys/tokens regularly.
- Enforce secret scanning and push protection.
- Use Dependabot/Renovate with controlled auto-merge rules.

---

## 9) Contributor Guidelines

### Proposing Changes
- Open focused PRs tied to one objective.
- Include context, risk assessment, and rollback notes.
- Provide evidence: test output, lint output, and relevant logs.

### Review Criteria
- Correctness: behavior meets acceptance criteria.
- Safety: no credential leaks, no privileged escalation, no unsafe defaults.
- Quality: lint/test pass; code is maintainable and observable.
- Scope: only intended files and functions changed.

### Validation Process
- CI must pass required checks.
- At least one domain reviewer (strategy/risk/infrastructure as relevant).
- Release-impacting changes require explicit versioning confirmation.

---

## 10) Troubleshooting and Optimization

### Common Issues and Fixes
- **Flaky tests**: add deterministic fixtures; mock unstable network edges.
- **CI slowdowns**: enable dependency + build caching; split test matrix.
- **Lint drift**: pin formatter/linter versions and run in pre-commit.
- **Async race bugs**: enforce bounded retries, locks/queues, and idempotent handlers.
- **Noisy alerts**: tune thresholds and add multi-signal correlation.

### Optimization Tips
- Parallelize independent jobs (lint/test/security).
- Run quick checks first for faster feedback.
- Track DORA-style and reliability metrics to guide process improvements.

---

## 11) Guide Maintenance Schedule

- **Weekly**: review CI failures, flaky tests, and dependency risk.
- **Monthly**: revise coding standards and assistant instructions.
- **Quarterly**: validate release/version strategy and security controls.
- **On major architecture change**: update this guide immediately.

Ownership recommendation:
- Platform/DevEx maintains CI/CD and AI policies.
- Strategy and risk leads approve domain-critical standards.

---

## 12) Closing Principle

The goal of this guide is to **standardize excellence** across the Trading Bot Swarm ecosystem—strengthening reliability, performance, and operational safety while enabling fast, high-confidence delivery.
