```markdown
# github-action-app Development Patterns

> Auto-generated skill from repository analysis

## Overview

This TypeScript-based GitHub Action application focuses on collaborative documentation workflows with AI assistants. The codebase emphasizes iterative documentation refinement and co-authored content creation, making it ideal for projects that require high-quality documentation maintained through AI-human collaboration.

## Coding Conventions

### File Naming
- Use **kebab-case** for all files
- Example: `copilot-codex-configuration-guide.md`, `github-action-app.ts`

### Imports and Exports
- **Mixed import style** - adapt to context:
  ```typescript
  // Named imports for utilities
  import { someFunction, anotherUtil } from './utils';
  
  // Default imports for main modules
  import mainModule from './main-module';
  ```

- **Mixed export style** - choose based on module purpose:
  ```typescript
  // Named exports for utilities
  export const helperFunction = () => {};
  export const configObject = {};
  
  // Default exports for main components
  export default class GitHubAction {}
  ```

### Commit Messages
- Use conventional prefixes: `docs:`, `feat:`
- Keep messages concise (~46 characters average)
- Examples:
  - `docs: update configuration guide`
  - `feat: add collaborative workflow support`

## Workflows

### Documentation Collaborative Update
**Trigger:** When documentation needs refinement or updates
**Command:** `/update-docs`

1. Create or identify documentation file in `docs/` or root directory
2. Make initial documentation creation/update commit
3. Request AI assistant suggestions for improvement
4. Create co-authored commit incorporating AI feedback:
   ```
   docs: enhance configuration guide
   
   Co-authored-by: Claude <claude@anthropic.com>
   ```
5. Optional: Create pull request for team review before merge

**Files typically involved:**
- `docs/*.md` files
- `CLAUDE.md`
- Root-level documentation files

### Iterative Documentation Refinement
**Trigger:** When documentation requires multiple iterations to reach final state
**Command:** `/refine-docs`

1. Create initial documentation commit with basic structure
2. Make multiple incremental updates based on:
   - User feedback
   - Technical accuracy review
   - Style and clarity improvements
3. Apply co-authored refinements with AI assistant
4. Perform final merge once documentation meets quality standards

**Example refinement cycle:**
```bash
# Initial commit
git commit -m "docs: initial draft of configuration guide"

# Incremental updates
git commit -m "docs: clarify setup instructions"
git commit -m "docs: add troubleshooting section"

# Co-authored final polish
git commit -m "docs: final refinements and formatting

Co-authored-by: Claude <claude@anthropic.com>"
```

## Testing Patterns

- Test files follow `*.test.*` pattern
- Framework: To be determined based on project needs
- Suggested structure:
  ```typescript
  // example.test.ts
  describe('Documentation Workflow', () => {
    it('should validate markdown syntax', () => {
      // Test implementation
    });
  });
  ```

## Commands

| Command | Purpose |
|---------|---------|
| `/update-docs` | Initiate collaborative documentation update workflow |
| `/refine-docs` | Start iterative documentation refinement process |
| `/review-commits` | Review commit message patterns and suggest improvements |
| `/check-conventions` | Validate file naming and code style conventions |
```