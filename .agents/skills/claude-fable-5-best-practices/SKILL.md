```markdown
# claude-fable-5-best-practices Development Patterns

> Auto-generated skill from repository analysis

## Overview
This skill teaches best practices for developing TypeScript projects in the `claude-fable-5-best-practices` repository. It covers file naming conventions, import/export styles, commit patterns, and testing approaches. The repository does not use a specific framework, focusing on clean, maintainable TypeScript code with consistent conventions.

## Coding Conventions

### File Naming
- Use **PascalCase** for filenames.
  - Example: `UserProfile.ts`, `DataFetcher.ts`

### Import Style
- Use **relative imports** for referencing modules.
  - Example:
    ```typescript
    import { fetchData } from './DataFetcher';
    ```

### Export Style
- Use **named exports** instead of default exports.
  - Example:
    ```typescript
    // In DataFetcher.ts
    export function fetchData() { /* ... */ }

    // In another file
    import { fetchData } from './DataFetcher';
    ```

### Commit Patterns
- Commit messages are freeform, with no strict prefixing.
- Average commit message length is around 59 characters.
  - Example:  
    ```
    Fix bug in data fetching logic for user profiles
    ```

## Workflows

### Code Development
**Trigger:** When adding or updating TypeScript code.
**Command:** `/develop-code`

1. Create a new file using PascalCase (e.g., `NewFeature.ts`).
2. Write code using named exports.
3. Use relative imports for dependencies.
4. Write or update corresponding test files (`*.test.ts`).

### Code Review
**Trigger:** When reviewing code for consistency and quality.
**Command:** `/review-code`

1. Check that filenames use PascalCase.
2. Ensure all imports are relative.
3. Verify that exports are named.
4. Confirm that new/updated code has corresponding tests.

### Testing
**Trigger:** When validating code changes.
**Command:** `/run-tests`

1. Locate test files matching the `*.test.*` pattern.
2. Run tests using the project's test runner (framework unknown).
3. Review test results and address any failures.

## Testing Patterns

- Test files follow the `*.test.*` naming convention (e.g., `UserProfile.test.ts`).
- The specific testing framework is unknown, but tests are colocated with source files or in a dedicated test directory.
- Example test file:
  ```typescript
  // UserProfile.test.ts
  import { getUserProfile } from './UserProfile';

  describe('getUserProfile', () => {
    it('returns the correct user data', () => {
      // test implementation
    });
  });
  ```

## Commands
| Command        | Purpose                                      |
|----------------|----------------------------------------------|
| /develop-code  | Start developing or updating TypeScript code  |
| /review-code   | Review code for style and convention         |
| /run-tests     | Run the test suite                           |
```