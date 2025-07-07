name: "Base PRP Template v2 - Context-Rich with Validation Loops"
description: |

## Purpose
Template optimized for AI agents to implement features with sufficient context and self-validation capabilities to achieve working code through iterative refinement.

## Core Principles
1. **Context is King**: Include ALL necessary documentation, examples, and caveats
2. **Validation Loops**: Provide executable tests/lints the AI can run and fix
3. **Information Dense**: Use keywords and patterns from the codebase
4. **Progressive Success**: Start simple, validate, then enhance
5. **Global rules**: Be sure to follow all rules in CLAUDE.md

---

## Goal
[What needs to be built - be specific about the end state and desires]

## Why
- [Business value and user impact]
- [Integration with existing features]
- [Problems this solves and for whom]

## What
[User-visible behavior and technical requirements]

### Success Criteria
- [ ] [Specific measurable outcomes]

## All Needed Context

### Documentation & References (list all context needed to implement the feature)
```yaml
# MUST READ - Include these in your context window
- url: [Official API docs URL]
  why: [Specific sections/methods you'll need]
  
- file: [path/to/example.ts]
  why: [Pattern to follow, gotchas to avoid]
  
- doc: [Library documentation URL] 
  section: [Specific section about common pitfalls]
  critical: [Key insight that prevents common errors]

- docfile: [PRPs/ai_docs/file.md]
  why: [docs that the user has pasted in to the project]

```

### Current Codebase tree (run `tree` in the root of the project) to get an overview of the codebase
```bash

```

### Desired Codebase tree with files to be added and responsibility of file
```bash

```

### Known Gotchas of our codebase & Library Quirks
```typescript
// CRITICAL: [Library name] requires [specific setup]
// Example: Express requires async functions for endpoints
// Example: MongoDB doesn't support transactions across collections
// Example: We use zod v3 and  
```

## Implementation Blueprint

### Data models and structure

Create the core data models, we ensure type safety and consistency.
```typescript
Examples: 
 - zod schemas (use snake_case for DB fields)
 - TypeScript interfaces (use camelCase for TS variables)
 - TypeScript types
 - zod validators
 - Drizzle schemas for MongoDB

```

### list of tasks to be completed to fullfill the PRP in the order they should be completed

```yaml
Task 1:
MODIFY src/existing-module.ts:
  - FIND pattern: "class OldImplementation"
  - INJECT after line containing "constructor"
  - PRESERVE existing method signatures

CREATE src/new-feature.ts:
  - MIRROR pattern from: src/similar-feature.ts
  - MODIFY class name and core logic
  - KEEP error handling pattern identical

...(...)

Task N:
...

```


### Per task pseudocode as needed added to each task
```typescript

// Task 1
// Pseudocode with CRITICAL details dont write entire code
async function newFeature(param: string): Promise<Result> {
  // PATTERN: Always validate input first (see src/validators.ts)
  const validated = validateInput(param);  // throws ValidationError
  
  // GOTCHA: MongoDB requires connection pooling
  const client = await getMongoClient();  // see src/db/connection.ts
  try {
    // PATTERN: Use existing retry decorator
    const result = await retry(3, exponentialBackoff, async () => {
      // CRITICAL: API returns 429 if >10 req/sec
      await rateLimiter.acquire();
      return await externalApi.call(validated);
    });
    
    // PATTERN: Standardized response format
    return formatResponse(result);  // see src/utils/responses.ts
  } finally {
    await client.close();
  }
}
```

### Integration Points
```yaml
DATABASE:
  - migration: "Add field 'feature_enabled' to users collection"
  - index: "CREATE INDEX idx_feature_lookup ON users(feature_id)"
  
CONFIG:
  - add to: config/settings.ts
  - pattern: "FEATURE_TIMEOUT = parseInt(process.env.FEATURE_TIMEOUT || '30')"
  
ROUTES:
  - add to: src/api/routes.ts  
  - pattern: "router.use('/feature', featureRouter)"
```

## Validation Loop

### Level 1: Syntax & Style
```bash
# Run these FIRST - fix any errors before proceeding
eslint src/new-feature.ts --fix  # Auto-fix what's possible
tsc --noEmit src/new-feature.ts  # Type checking

# Expected: No errors. If errors, READ the error and fix.
```

### Level 2: Unit Tests each new feature/file/function use existing test patterns
```typescript
// CREATE test_new-feature.test.ts with these test cases:
import { test, describe } from 'node:test';
import assert from 'node:assert';

describe('newFeature', () => {
  test('basic functionality works', async () => {
    const result = await newFeature("valid_input");
    assert.strictEqual(result.status, "success");
  });

  test('invalid input throws ValidationError', async () => {
    await assert.rejects(async () => {
      await newFeature("");
    }, ValidationError);
  });

  test('handles timeouts gracefully', async () => {
    // Mock external API to throw timeout error
    const originalCall = externalApi.call;
    externalApi.call = async () => { throw new Error('timeout'); };
    
    try {
      const result = await newFeature("valid");
      assert.strictEqual(result.status, "error");
      assert(result.message.includes("timeout"));
    } finally {
      externalApi.call = originalCall;
    }
  });
});
```

```bash
# Run and iterate until passing:
pnpm test test_new-feature.test.ts
# If failing: Read error, understand root cause, fix code, re-run (never mock to pass)
```

### Level 3: Integration Test
```bash
# Start the service
pnpm run dev

# Test the endpoint
curl -X POST http://localhost:3000/feature \
  -H "Content-Type: application/json" \
  -d '{"param": "test_value"}'

# Expected: {"status": "success", "data": {...}}
# If error: Check logs at logs/app.log for stack trace
```

## Final validation Checklist
- [ ] All tests pass: `pnpm test`
- [ ] No linting errors: `eslint src/`
- [ ] No type errors: `tsc --noEmit`
- [ ] Manual test successful: [specific curl/command]
- [ ] Error cases handled gracefully
- [ ] Logs are informative but not verbose
- [ ] Documentation updated if needed

---

## Anti-Patterns to Avoid
- ❌ Don't create new patterns when existing ones work
- ❌ Don't skip validation because "it should work"  
- ❌ Don't ignore failing tests - fix them
- ❌ Don't use sync functions in async context
- ❌ Don't hardcode values that should be config
- ❌ Don't catch all exceptions - be specific