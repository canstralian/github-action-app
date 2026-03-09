# CLAUDE.md — AI Assistant Guide for github-action-app

This file provides context for AI coding assistants (Claude, Copilot, etc.) working on this repository. It describes the project purpose, structure, development workflows, and conventions to follow.

---

## Project Overview

**Repository:** `canstralian/github-action-app`
**Purpose:** A GitHub Actions application — tooling, workflows, or an app that integrates with the GitHub Actions platform.

This project is in its initial state. Update this file as the codebase evolves.

---

## Repository Structure

```
github-action-app/
└── CLAUDE.md              # This file — AI assistant guide
```

> This project is in its bootstrapping phase. The structure above reflects the current state. Update this section as files and directories are added.

---

## Development Workflows

### Branching Strategy

- `main` — stable, production-ready code. Never push directly.
- `claude/<description>-<session-id>` — branches created by AI assistants.
- `feat/<description>` — feature branches created by humans.
- `fix/<description>` — bug fix branches.

All work must go through pull requests. Branch names must be descriptive and lowercase with hyphens.

### Making Changes

1. Check out or create a branch from `main`.
2. Make focused, minimal changes — do not refactor code unrelated to the task.
3. Write or update tests to cover the change.
4. Run linting and tests locally before committing.
5. Write a clear commit message (see conventions below).
6. Push the branch and open a pull request.

### Running Tests

```bash
# Adapt to the actual test framework used
npm test          # Node.js / Jest / Vitest
pytest            # Python
go test ./...     # Go
```

### Linting & Formatting

```bash
# Adapt to tooling in use
npm run lint      # ESLint / Prettier
npm run format
ruff check .      # Python
gofmt ./...       # Go
```

---

## Commit Message Conventions

Follow the **Conventional Commits** specification:

```
<type>(<scope>): <short summary>

[optional body]

[optional footer]
```

**Types:**
| Type       | When to use                              |
|------------|------------------------------------------|
| `feat`     | New feature or capability                |
| `fix`      | Bug fix                                  |
| `docs`     | Documentation only                       |
| `refactor` | Code restructuring without behavior change |
| `test`     | Adding or updating tests                 |
| `chore`    | Tooling, dependencies, config            |
| `ci`       | Changes to GitHub Actions workflows      |

**Examples:**
```
feat(action): add support for matrix build strategy
fix(workflow): correct permissions for GITHUB_TOKEN
docs: update CLAUDE.md with project structure
ci: add caching step for Node modules
```

---

## GitHub Actions Conventions

When working with workflow files in `.github/workflows/`:

- Use `actions/checkout@v4` (or latest stable) — do not pin to old versions without reason.
- Always set explicit `permissions` blocks — follow the principle of least privilege.
- Prefer `ubuntu-latest` runners unless a specific OS is required.
- Use `${{ secrets.* }}` for sensitive values — never hardcode credentials.
- Cache dependencies to speed up workflows (e.g., `actions/cache`, `setup-node` built-in cache).
- Use `concurrency` groups to cancel redundant workflow runs on the same branch.
- Pin third-party actions to a full commit SHA for security, with a version comment:
  ```yaml
  uses: some-org/some-action@abc1234  # v2.3.1
  ```

---

## Code Conventions

### General

- Keep functions small and single-purpose.
- Prefer explicit over implicit — avoid magic values; use named constants.
- Do not add comments for self-evident code. Add comments only where intent is non-obvious.
- Do not add error handling for scenarios that cannot occur.
- Do not engineer for hypothetical future requirements — YAGNI.

### Security

- Never commit secrets, tokens, or credentials.
- Validate all inputs at system boundaries (user input, external APIs, workflow inputs).
- Follow OWASP Top 10 guidance.
- For GitHub Actions: restrict `GITHUB_TOKEN` permissions; audit third-party action usage.

### Testing

- Write tests alongside new functionality — do not defer test writing.
- Unit tests for pure logic; integration tests for workflow/API behavior.
- Test files mirror the source structure (e.g., `tests/unit/` vs `src/`).

---

## AI Assistant Instructions

When working in this repository as an AI assistant:

1. **Read before editing.** Always read the relevant file(s) before proposing or making changes.
2. **Minimal changes.** Only modify what is necessary to complete the task. Do not clean up unrelated code.
3. **No speculative additions.** Do not add features, configuration, or handling for scenarios not in scope.
4. **No new files unless necessary.** Prefer editing existing files over creating new ones.
5. **Follow the branching strategy.** Develop on the designated `claude/` branch; never push to `main`.
6. **Commit and push.** After completing a task, commit with a clear message and push to the feature branch.
7. **Update CLAUDE.md.** If you discover new conventions, structures, or workflows, update this file.
8. **Security first.** Never introduce vulnerabilities — command injection, hardcoded secrets, overly broad permissions, etc.

---

## Key Files to Know

| File/Path | Purpose |
|-----------|---------|
| `CLAUDE.md` | This guide — AI assistant context |

> Expand this table as significant files are added to the project.

---

## Getting Help

- Open an issue on GitHub for bugs or feature requests.
- Reference this file when onboarding new contributors or AI sessions.
- Last updated: 2026-03-09
