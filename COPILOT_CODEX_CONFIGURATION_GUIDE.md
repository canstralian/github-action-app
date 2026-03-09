# Trading Bot Swarm: GitHub Copilot + Codex Configuration Guide

## 1) Purpose and Scope

This guide standardizes how contributors configure and use **GitHub Copilot** and **Codex** across the Trading Bot Swarm ecosystem. The goal is to keep generated code consistent, testable, secure, and production-ready.

Copilot should be treated as a **pair programmer** with strict behavioral rules:
- It can suggest implementation details, but humans remain accountable for architecture and security decisions.
- It must follow project standards for testing, linting, typing, and security controls.
- It must not bypass validation gates or introduce shortcuts that weaken reliability.

This guide applies to:
- Strategy services and execution engines.
- Shared libraries and tooling.
- CI/CD automation and release pipelines.
- Security and dependency governance.

---

## 2) Configuration Overview

### Testing and Quality Gates
- Require tests for every code change that affects behavior.
- Allow docs-only changes to skip test execution when code paths are untouched.
- Enforce minimum quality gates in CI before merge:
  - Unit tests
  - Integration tests (where applicable)
  - Linting
  - Type checks

### Linting and Code Style
- Adopt a single source of truth for formatting and linting per language (for example: ESLint + Prettier for TypeScript, Ruff + Black for Python).
- Run formatters in CI to detect drift.
- Keep style auto-fixable when possible.

### Async Patterns and Reliability
- Use explicit timeouts and retries for external API calls.
- Implement circuit-breaker or backoff patterns in trading-critical flows.
- Avoid fire-and-forget operations for order placement, risk checks, or position reconciliation.

### Security Defaults
- Default to least privilege for GitHub tokens and cloud identities.
- Forbid hardcoded credentials and API keys.
- Redact secrets in logs and error payloads.
- Pin third-party GitHub Actions versions (prefer commit SHA pinning).

### Logging and Observability
- Standardize structured logs (JSON preferred).
- Include correlation IDs, strategy IDs, and execution context in request/transaction logs.
- Emit metrics for:
  - Order placement latency
  - API error rate
  - Fill/reject ratios
  - Risk rule violations

### CI/CD Integration
- Run lint + test on pull requests and protected branches.
- Add branch protections requiring passing checks and at least one reviewer.
- Use separate deployment environments (dev/staging/prod) with approvals for production.

### Version Control Expectations
- Use conventional commits (`feat:`, `fix:`, `chore:`, etc.).
- Keep PRs small and scoped.
- Require review sign-off for strategy logic, risk controls, and execution code.

---

## 3) Custom Instruction Behavior (Codex + Copilot)

Below is an example **conceptual** YAML for organization-level instruction behavior.

```yaml
assistant_policy:
  role: "pair_programmer"
  principles:
    - "Prefer minimal, focused diffs."
    - "Do not modify unrelated files."
    - "Never add hardcoded secrets."
    - "Always preserve deterministic behavior in trading-critical paths."

copilot:
  instruction_rules:
    - id: test-required-for-code
      when: "code_changed == true"
      require:
        - "run_unit_tests"
        - "run_linter"
        - "run_type_checks"
    - id: docs-only-skip
      when: "only_docs_changed == true"
      allow:
        - "skip_tests"
        - "skip_lint"
      note: "Still run markdown lint if enabled."
    - id: security-constraints
      deny:
        - "hardcoded_credentials"
        - "overly_broad_token_permissions"
        - "unsafe_shell_injection_patterns"

codex:
  instruction_rules:
    - id: review-before-edit
      require:
        - "read_target_files_before_patch"
    - id: quality-gates
      require:
        - "execute_repo_checks_before_commit"
    - id: observability
      require:
        - "add_or_preserve_structured_logging"
        - "preserve_error_context_without_secret_leakage"
```

### Example Rule Set (Human-Readable)
- If code changes are made, run relevant tests and linters before commit.
- If only documentation is changed, skip code tests and linters.
- Never weaken risk checks or validation logic without explicit design approval.
- Prefer explicit error handling over silent failures in execution and reconciliation flows.

---

## 4) GitHub Workflow Example: Lint + Test Automation

Example workflow (`.github/workflows/quality-gate.yml`):

```yaml
name: quality-gate

on:
  pull_request:
    branches: [main, develop]
    paths-ignore:
      - "**/*.md"
      - "docs/**"
  push:
    branches: [main, develop]

permissions:
  contents: read

concurrency:
  group: quality-gate-${{ github.ref }}
  cancel-in-progress: true

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup runtime
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type Check
        run: npm run typecheck --if-present

      - name: Unit Test
        run: npm test -- --ci
```

### Trigger and Job Conditions
- Trigger on pull requests to `main`/`develop`.
- Ignore docs-only changes for code quality gates.
- Trigger on direct pushes to protected branches.
- Quality job sequence:
  1. Checkout
  2. Runtime setup + cache
  3. Dependency install
  4. Lint
  5. Type check
  6. Unit test

---

## 5) Semantic Release and Version Tagging Best Practices

Use semantic-release for automated versioning and changelog generation.

```yaml
name: release

on:
  push:
    branches: [main]

permissions:
  contents: write
  issues: write
  pull-requests: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"
      - run: npm ci
      - run: npx semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

Versioning recommendations:
- `feat:` → minor bump.
- `fix:` → patch bump.
- `BREAKING CHANGE:` footer → major bump.
- Protect `main` so only validated commits can trigger release.

---

## 6) Security and Dependency Scanning Best Practices

Example security workflow (`.github/workflows/security-scan.yml`):

```yaml
name: security-scan

on:
  pull_request:
  schedule:
    - cron: "0 3 * * 1"

permissions:
  contents: read
  security-events: write

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Dependency Audit
        run: npm audit --audit-level=high

      - name: CodeQL Init
        uses: github/codeql-action/init@v3
        with:
          languages: javascript

      - name: CodeQL Analyze
        uses: github/codeql-action/analyze@v3
```

Additional recommendations:
- Enable Dependabot updates for package managers and GitHub Actions.
- Treat critical vulnerabilities as release blockers.
- Maintain an exception process with owner approval and expiration dates.

---

## 7) Contributor Guidelines

### Proposing Changes
- Open a focused PR with clear scope.
- Include risk impact notes for strategy/execution/risk modules.
- Link test evidence and CI results.

### Review Criteria
- Correctness and deterministic behavior under failure conditions.
- Test coverage quality for changed logic.
- Security posture (secret handling, permissions, validation).
- Operational readiness (logs, metrics, alerts).

### Validation Process
- Local checks pass (lint/test/type/security where applicable).
- CI quality gates pass.
- Reviewer confirms no regression risk in trading-critical paths.

---

## 8) Troubleshooting and Optimization

### Common Issues
- **Flaky tests**: add deterministic test fixtures and mock external market APIs.
- **Slow CI**: enable dependency caching and parallelize independent jobs.
- **False-positive security alerts**: document suppression rationale and expiration.
- **Instruction drift**: centralize assistant instructions and version them.

### Optimization Tips
- Keep assistant instructions concise and explicit.
- Use templates for PR descriptions and post-merge validation steps.
- Periodically audit generated code quality against incident reports.

---

## 9) Maintenance Schedule

Review this guide:
- **Monthly** for tooling/version drift.
- **Quarterly** for policy and security alignment.
- **Immediately** after incidents, major architecture changes, or compliance updates.

Ownership recommendations:
- Engineering Productivity: CI/CD and lint/test standards.
- Security Team: scanning, secrets policy, dependency governance.
- Trading Platform Leads: strategy safety, reliability, and observability standards.

---

## 10) Closing Note

This standard exists to institutionalize engineering excellence and strengthen the Trading Bot Swarm ecosystem’s reliability, performance, and safety. By aligning Copilot and Codex behavior with strict quality and security rules, teams can move quickly without compromising trust in automated trading operations.
