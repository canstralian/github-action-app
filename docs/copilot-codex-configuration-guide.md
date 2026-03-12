# Copilot + Codex Configuration Guide for Trading Bot Swarm

## 1) Purpose and Scope

This guide standardizes how GitHub Copilot and Codex are configured and used across the Trading Bot Swarm ecosystem.

The goal is to ensure that AI-assisted development is:

- consistent across repositories and teams,
- enforceable through automation,
- aligned with secure-by-default engineering,
- auditable through CI/CD quality gates.

Copilot should be treated as a **pair programmer**, not an autonomous deployer. It can draft, suggest, and refactor code, but it must follow strict behavioral rules around testing, linting, security, and reviewability.

---

## 2) Configuration Overview

Use these baseline standards in every bot or shared service repository.

### Testing

- Require tests for all functional code changes.
- Run full test suites in CI on pull requests and pushes to protected branches.
- Permit documentation-only changes to skip test execution.

### Linting and Formatting

- Enforce language-specific lint rules (for example: Ruff/Flake8, ESLint, Clippy).
- Run formatter checks in CI (`--check` mode).
- Block merge on lint failures.

### Code Style

- Use explicit types/interfaces where practical.
- Prefer deterministic logic for strategy-critical code paths.
- Keep functions focused and side effects explicit.

### Async Patterns

- Require timeouts and cancellation handling for network operations.
- Avoid unbounded retries and uncontrolled parallelism.
- Use structured concurrency primitives when available.

### Security Defaults

- Never commit secrets, API keys, or exchange credentials.
- Use least-privilege tokens for automation.
- Require dependency and secret scanning in CI.
- Gate high-risk changes (execution engine, order routing, wallet transfers) behind human approval.

### Logging and Observability

- Log with structured fields (`strategy`, `symbol`, `order_id`, `trace_id`).
- Redact sensitive values in all logs.
- Emit metrics for latency, error rate, and fill quality.
- Attach trace identifiers to events crossing services.

### CI/CD Integration

- Enforce lint + test quality gates before merge.
- Use branch protection requiring successful checks.
- Automate versioning and changelog publication.

### Version Control

- Use short-lived feature branches.
- Prefer small, reviewable pull requests.
- Use conventional commit messages for release automation.

---

## 3) Custom Instruction Behavior for Codex and Copilot

Both assistants should follow repository-level custom instructions that define behavior boundaries.

### Example Rule Set

- Generate code that passes lint and tests.
- For non-doc code changes, run affected tests and lints before proposing final output.
- Do not modify unrelated files.
- Prefer secure and deterministic patterns over clever shortcuts.
- For docs-only diffs, skip test execution and note skip reason.

### Conceptual Custom Instructions (YAML)

```yaml
assistant_policy:
  agents:
    copilot:
      role: "pair_programmer"
      behavior:
        - "suggest_small_reviewable_changes"
        - "prefer_existing_patterns"
        - "never_bypass_security_controls"
    codex:
      role: "implementation_agent"
      behavior:
        - "read_repo_instructions_before_editing"
        - "run_tests_and_linters_for_code_changes"
        - "skip_tests_for_docs_only_changes_with_reason"

quality_gates:
  require_for_code_changes:
    - lint
    - unit_tests
    - integration_tests_if_touched
  docs_only_exemptions:
    - unit_tests
    - integration_tests

security:
  defaults:
    secret_scanning: true
    dependency_scanning: true
    least_privilege_tokens: true
  require_human_approval_for:
    - "security-sensitive"
    - "high-blast-radius"
    - "ambiguous-intent"

engineering_standards:
  async:
    require_timeout: true
    require_cancellation: true
    max_retry_policy: "bounded_exponential"
  observability:
    structured_logging: true
    redact_secrets: true
    trace_propagation: true
```

---

## 4) GitHub Workflow Example: Lint + Test Automation

Use a pull-request quality gate workflow to enforce baseline checks.

### Trigger Conditions

- `pull_request` on `main` and `develop`
- `push` on `main` (optional for post-merge verification)
- Ignore docs-only updates where policy allows

### Quality Gate Job Steps

1. Checkout source
2. Setup runtime(s)
3. Install dependencies
4. Run lint checks
5. Run tests
6. Upload test artifacts (if applicable)

```yaml
name: quality-gate

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: true

permissions:
  contents: read

jobs:
  quality-gate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Test
        run: npm test -- --ci
```

---

## 5) Best-Practice Workflow: Semantic Release + Version Tagging

Use semantic versioning driven by conventional commits.

```yaml
name: release

on:
  push:
    branches: [main]

concurrency:
  group: release-${{ github.ref }}
  cancel-in-progress: false

permissions:
  contents: write
  issues: write
  pull-requests: write

jobs:
  semantic-release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Install
        run: npm ci

      - name: Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
        run: npx semantic-release
```

---

## 6) Best-Practice Workflow: Security + Dependency Scanning

Run scheduled and pull-request scans to detect vulnerabilities early.

```yaml
name: security-scan

on:
  pull_request:
    branches: [main, develop]
  schedule:
    - cron: "0 6 * * 1"

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: true

permissions:
  contents: read
  security-events: write

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Dependency audit
        run: npm audit --audit-level=high

      - name: CodeQL Init
        uses: github/codeql-action/init@v3
        with:
          languages: javascript-typescript

      - name: CodeQL Analyze
        uses: github/codeql-action/analyze@v3
```

---

## 7) Contributor Guidelines

### Proposing Changes

- Keep PRs focused on one intent.
- Include risk notes for strategy or execution-path changes.
- Provide test evidence (output snippets or CI links).

### Review Criteria

- Code correctness and deterministic behavior.
- Security implications and privilege boundaries.
- Observability quality (logs, metrics, traces).
- Operational rollback clarity.

### Validation Process

- Local lint + tests before opening PR.
- CI checks must pass.
- Human review required for high-risk domains.

---

## 8) Troubleshooting and Optimization

### Common Issues

- **Flaky tests:** isolate time-dependent tests; use deterministic fixtures.
- **Async deadlocks/timeouts:** ensure every async path has timeout + cancellation.
- **Lint drift across repos:** centralize shared config in reusable templates.
- **Slow CI:** enable dependency caching and split jobs by concern.

### Optimization Tips

- Introduce test impact analysis for large monorepos.
- Maintain reusable GitHub Actions for common quality gates.
- Track DORA and defect escape metrics to tune policy strictness.

---

## 9) Guide Maintenance Schedule

- **Weekly:** review CI failures and false positives; refine lint/test rules.
- **Monthly:** update dependency and security scanning policies.
- **Quarterly:** review Copilot/Codex instruction quality and rule compliance.
- **Release-cycle checkpoints:** verify workflows match current runtime/toolchain.

Assign ownership to platform engineering (or DevEx) and require changelog notes for policy adjustments.

---

## 10) Closing Note

This guide exists to standardize excellence in AI-assisted software delivery for Trading Bot Swarm.

By aligning Copilot and Codex behavior with strict quality and security controls, we strengthen the **reliability, performance, and safety** of the trading ecosystem.
