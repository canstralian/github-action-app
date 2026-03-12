# github-action-app Development Patterns

> Auto-generated skill from repository analysis

## Overview

This TypeScript-based GitHub Action application follows collaborative development patterns with a strong emphasis on documentation-driven development. The codebase demonstrates modern AI-assisted workflows where multiple AI assistants contribute to documentation and code refinement through co-authored commits.

## Coding Conventions

### File Naming
Use kebab-case for all files:
```
action.yml
package-lock.json
some-feature.ts
my-component.test.ts
```

### Import/Export Style
The codebase uses mixed import and export patterns. Follow these guidelines:

**Imports:**
```typescript
// ES6 imports for modules
import { someFunction } from './utils'
import * as core from '@actions/core'

// CommonJS for Node.js modules when needed
const fs = require('fs')
```

**Exports:**
```typescript
// Named exports (preferred)
export const myFunction = () => { }
export { anotherFunction }

// Default exports for main modules
export default class MyClass { }
```

### Commit Message Format
- Use conventional commit prefixes: `feat:`, `docs:`, `fix:`
- Keep messages concise (average 46 characters)
- Examples:
  - `docs: update README with new examples`
  - `feat: add error handling for edge cases`

## Workflows

### AI Collaborative Documentation
**Trigger:** When documentation needs refinement or AI assistants suggest improvements
**Command:** `/ai-review-docs`

1. Create or update initial documentation in `docs/*.md` or root markdown files
2. AI assistant reviews and suggests improvements via co-authored commit
3. Additional AI assistants may contribute sequentially for further refinement
4. Each iteration improves clarity, accuracy, and completeness
5. Final review ensures consistency across all documentation

**Files typically involved:**
- `docs/*.md`
- `CLAUDE.md`
- `README.md`

**Example workflow:**
```bash
# Initial documentation
git add docs/new-feature.md
git commit -m "docs: add initial feature documentation"

# AI assistant suggests improvements
# (AI creates co-authored commit with refinements)
git commit -m "docs: refine feature documentation

Co-authored-by: GitHub-Copilot <github-copilot@github.com>"
```

### Documentation Iteration
**Trigger:** When documentation requires multiple refinements or corrections
**Command:** `/refine-docs`

1. Create initial documentation commit
2. Review documentation for clarity and completeness
3. Make targeted updates to specific sections
4. Commit incremental improvements
5. Repeat until documentation meets quality standards
6. Final polish and consistency check

**Files typically involved:**
- All `*.md` files
- Documentation in `docs/` directory

**Example iteration:**
```typescript
// First commit: Basic structure
// Second commit: Add examples
// Third commit: Fix formatting and typos
// Fourth commit: Add cross-references
```

## Testing Patterns

### Test File Structure
- Test files follow the pattern: `*.test.*`
- Place tests adjacent to source files or in dedicated test directories
- Use descriptive test names that explain the behavior being tested

### Testing Guidelines
```typescript
// Example test structure (framework agnostic)
describe('Feature Name', () => {
  test('should handle expected input correctly', () => {
    // Arrange
    const input = 'test data'
    
    // Act
    const result = myFunction(input)
    
    // Assert
    expect(result).toBe('expected output')
  })
})
```

## Commands

| Command | Purpose |
|---------|---------|
| `/ai-review-docs` | Initiate AI collaborative documentation review |
| `/refine-docs` | Start iterative documentation improvement process |
| `/lint-code` | Check TypeScript code style and conventions |
| `/test-changes` | Run tests for modified files |
| `/format-commits` | Ensure commit messages follow conventional format |

## Best Practices

1. **Documentation First**: Write or update documentation before implementing features
2. **Iterative Refinement**: Don't aim for perfection in first draft - iterate based on feedback
3. **AI Collaboration**: Leverage AI assistants for reviews and suggestions
4. **Consistent Naming**: Stick to kebab-case for files and follow TypeScript naming conventions
5. **Small Commits**: Keep commits focused and atomic for better collaboration