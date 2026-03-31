# github-action-app Development Patterns

> Auto-generated skill from repository analysis

## Overview

This skill covers development patterns for a TypeScript-based GitHub Action application. The codebase follows modern TypeScript practices with a focus on clean documentation and iterative development workflows. The project emphasizes documentation-driven development with regular refinement cycles.

## Coding Conventions

### File Naming
Use kebab-case for all files:
```
action-handler.ts
config-parser.ts
utils/string-helpers.ts
docs/getting-started.md
```

### Import/Export Patterns
The codebase uses mixed import and export styles. Follow these guidelines:

**Imports:**
```typescript
// External dependencies
import * as core from '@actions/core'
import { getInput, setOutput } from '@actions/core'

// Internal modules
import { parseConfig } from './config-parser'
import * as utils from './utils'
```

**Exports:**
```typescript
// Named exports for utilities
export const processInput = (input: string) => { /* ... */ }
export { validateConfig } from './validators'

// Default exports for main modules
export default class ActionHandler { /* ... */ }
```

### Commit Message Style
Follow conventional commit format with concise messages (average 46 characters):
```
feat: add input validation
docs: update README examples
fix: handle edge case in parser
```

## Workflows

### Documentation Refinement
**Trigger:** When documentation needs to be created, updated, or refined
**Command:** `/refine-docs`

1. **Create initial documentation**
   - Start with basic structure and core content
   - Focus on getting the main points documented first
   - Use clear headings and consistent formatting

2. **Make iterative updates**
   - Review documentation for clarity and completeness
   - Add missing sections or examples
   - Update outdated information

3. **Apply AI assistant suggestions**
   - Use AI tools to improve writing quality
   - Enhance technical accuracy and completeness
   - Ensure consistent tone and style

4. **Merge pull request with final changes**
   - Review all changes for accuracy
   - Ensure documentation builds correctly
   - Merge when content meets quality standards

**Files typically involved:**
- `CLAUDE.md` - AI assistant documentation
- `docs/*.md` - General documentation files
- `README.md` - Project overview and setup

**Example workflow:**
```bash
# Create or update documentation
git checkout -b docs/refine-action-usage
# Edit documentation files
git add docs/
git commit -m "docs: refine action usage examples"
# Create PR for review and AI refinement
```

## Testing Patterns

Testing framework is flexible, but files should follow the `*.test.*` pattern:

```typescript
// action-handler.test.ts
describe('ActionHandler', () => {
  it('should process valid input', () => {
    // Test implementation
  })
  
  it('should handle invalid input gracefully', () => {
    // Test implementation
  })
})

// utils.test.js
test('string helper functions', () => {
  // Test implementation
})
```

## Commands

| Command | Purpose |
|---------|---------|
| `/refine-docs` | Trigger documentation refinement workflow |
| `/add-tests` | Add or update test files following project patterns |
| `/fix-imports` | Standardize import/export patterns across files |
| `/commit-conventional` | Create conventional commit message |