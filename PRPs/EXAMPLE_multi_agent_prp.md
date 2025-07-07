name: "Multi-Agent System: Research Agent with Email Draft Sub-Agent"
description: |

## Purpose
Build a TypeScript AI multi-agent system where a primary Research Agent uses Brave Search API and has an Email Draft Agent (using Gmail API) as a tool. This demonstrates agent-as-tool pattern with external API integrations.

## Core Principles
1. **Context is King**: Include ALL necessary documentation, examples, and caveats
2. **Validation Loops**: Provide executable tests/lints the AI can run and fix
3. **Information Dense**: Use keywords and patterns from the codebase
4. **Progressive Success**: Start simple, validate, then enhance

---

## Goal
Create a production-ready multi-agent system where users can research topics via CLI, and the Research Agent can delegate email drafting tasks to an Email Draft Agent. The system should support multiple LLM providers and handle API authentication securely.

## Why
- **Business value**: Automates research and email drafting workflows
- **Integration**: Demonstrates advanced TypeScript AI multi-agent patterns
- **Problems solved**: Reduces manual work for research-based email communications

## What
A CLI-based application where:
- Users input research queries
- Research Agent searches using Brave API
- Research Agent can invoke Email Draft Agent to create Gmail drafts
- Results stream back to the user in real-time

### Success Criteria
- [ ] Research Agent successfully searches via Brave API
- [ ] Email Agent creates Gmail drafts with proper authentication
- [ ] Research Agent can invoke Email Agent as a tool
- [ ] CLI provides streaming responses with tool visibility
- [ ] All tests pass and code meets quality standards

## All Needed Context

### Documentation & References
```yaml
# MUST READ - Include these in your context window
- url: https://ai.pydantic.dev/agents/
  why: Core agent creation patterns
  
- url: https://ai.pydantic.dev/multi-agent-applications/
  why: Multi-agent system patterns, especially agent-as-tool
  
- url: https://developers.google.com/gmail/api/guides/sending
  why: Gmail API authentication and draft creation
  
- url: https://api-dashboard.search.brave.com/app/documentation
  why: Brave Search API REST endpoints
  
- file: examples/agent/agent.ts
  why: Pattern for agent creation, tool registration, dependencies
  
- file: examples/agent/providers.ts
  why: Multi-provider LLM configuration pattern
  
- file: examples/cli.ts
  why: CLI structure with streaming responses and tool visibility

- url: https://github.com/googleworkspace/nodejs-samples/blob/main/gmail/snippet/send%20mail/create_draft.js
  why: Official Gmail draft creation example
```

### Current Codebase tree
```bash
.
├── examples/
│   ├── agent/
│   │   ├── agent.ts
│   │   ├── providers.ts
│   │   └── ...
│   └── cli.ts
├── PRPs/
│   └── templates/
│       └── prp_base.md
├── INITIAL.md
├── CLAUDE.md
└── package.json
```

### Desired Codebase tree with files to be added
```bash
.
├── agents/
│   ├── index.ts               # Package exports
│   ├── research_agent.ts      # Primary agent with Brave Search
│   ├── email_agent.ts         # Sub-agent with Gmail capabilities
│   ├── providers.ts           # LLM provider configuration
│   └── models.ts              # Zod schemas for data validation
├── tools/
│   ├── index.ts               # Package exports
│   ├── brave_search.ts        # Brave Search API integration
│   └── gmail_tool.ts          # Gmail API integration
├── config/
│   ├── index.ts               # Package exports
│   └── settings.ts            # Environment and config management
├── tests/
│   ├── agents/
│   │   ├── research_agent.test.ts   # Research agent tests
│   │   └── email_agent.test.ts      # Email agent tests
│   ├── tools/
│   │   ├── brave_search.test.ts     # Brave search tool tests
│   │   └── gmail_tool.test.ts       # Gmail tool tests
│   └── cli.test.ts            # CLI tests
├── cli.ts                     # CLI interface
├── .env.example               # Environment variables template
├── package.json               # Updated dependencies
├── README.md                  # Comprehensive documentation
└── credentials/.gitkeep       # Directory for Gmail credentials
```

### Known Gotchas & Library Quirks
```typescript
// CRITICAL: TypeScript AI requires async throughout - no sync functions in async context
// CRITICAL: Gmail API requires OAuth2 flow on first run - credentials.json needed
// CRITICAL: Brave API has rate limits - 2000 req/month on free tier
// CRITICAL: Agent-as-tool pattern requires passing ctx.usage for token tracking
// CRITICAL: Gmail drafts need base64 encoding with proper MIME formatting
// CRITICAL: Always use absolute imports for cleaner code
// CRITICAL: Store sensitive credentials in .env, never commit them
```

## Implementation Blueprint

### Data models and structure

```typescript
// models.ts - Core data structures
import { z } from 'zod';

export const ResearchQuerySchema = z.object({
  query: z.string().min(1, "Query cannot be empty"),
  maxResults: z.number().min(1).max(50).default(10),
  includeSummary: z.boolean().default(true)
});

export const BraveSearchResultSchema = z.object({
  title: z.string(),
  url: z.string().url(),
  description: z.string(),
  score: z.number().min(0).max(1).default(0)
});

export const EmailDraftSchema = z.object({
  to: z.array(z.string().email()).min(1, "At least one recipient required"),
  subject: z.string().min(1, "Subject cannot be empty"),
  body: z.string().min(1, "Body cannot be empty"),
  cc: z.array(z.string().email()).optional(),
  bcc: z.array(z.string().email()).optional()
});

export const ResearchEmailRequestSchema = z.object({
  researchQuery: z.string(),
  emailContext: z.string().min(1, "Email context required"),
  recipientEmail: z.string().email()
});

export type ResearchQuery = z.infer<typeof ResearchQuerySchema>;
export type BraveSearchResult = z.infer<typeof BraveSearchResultSchema>;
export type EmailDraft = z.infer<typeof EmailDraftSchema>;
export type ResearchEmailRequest = z.infer<typeof ResearchEmailRequestSchema>;
```

### List of tasks to be completed

```yaml
Task 1: Setup Configuration and Environment
CREATE config/settings.ts:
  - PATTERN: Use zod for environment validation like examples use dotenv
  - Load environment variables with defaults
  - Validate required API keys present

CREATE .env.example:
  - Include all required environment variables with descriptions
  - Follow pattern from examples/README.md

Task 2: Implement Brave Search Tool
CREATE tools/brave_search.ts:
  - PATTERN: Async functions like examples/agent/tools.ts
  - Simple REST client using axios (already in dependencies)
  - Handle rate limits and errors gracefully
  - Return structured BraveSearchResult models

Task 3: Implement Gmail Tool
CREATE tools/gmail_tool.ts:
  - PATTERN: Follow OAuth2 flow from Gmail quickstart
  - Store token.json in credentials/ directory
  - Create draft with proper MIME encoding
  - Handle authentication refresh automatically

Task 4: Create Email Draft Agent
CREATE agents/email_agent.ts:
  - PATTERN: Follow examples/agent/agent.ts structure
  - Use Agent with deps_type pattern
  - Register gmail_tool as @agent.tool
  - Return EmailDraft model

Task 5: Create Research Agent
CREATE agents/research_agent.ts:
  - PATTERN: Multi-agent pattern from TypeScript AI docs
  - Register brave_search as tool
  - Register email_agent.run() as tool
  - Use RunContext for dependency injection

Task 6: Implement CLI Interface
CREATE cli.ts:
  - PATTERN: Follow examples/cli.ts streaming pattern
  - Color-coded output with tool visibility
  - Handle async properly with async/await
  - Session management for conversation context

Task 7: Add Comprehensive Tests
CREATE tests/:
  - PATTERN: Mirror examples test structure
  - Mock external API calls
  - Test happy path, edge cases, errors
  - Ensure 80%+ coverage

Task 8: Create Documentation
CREATE README.md:
  - PATTERN: Follow examples/README.md structure
  - Include setup, installation, usage
  - API key configuration steps
  - Architecture diagram
```

### Per task pseudocode

```typescript
// Task 2: Brave Search Tool
async function searchBrave(query: string, apiKey: string, count: number = 10): Promise<BraveSearchResult[]> {
  // PATTERN: Use axios like examples use fetch
  const headers = { "X-Subscription-Token": apiKey };
  const params = { q: query, count };
  
  // GOTCHA: Brave API returns 401 if API key invalid
  const response = await axios.get(
    "https://api.search.brave.com/res/v1/web/search",
    { headers, params, timeout: 30000 }  // CRITICAL: Set timeout to avoid hanging
  );
  
  // PATTERN: Structured error handling
  if (response.status !== 200) {
    throw new Error(`Brave API returned ${response.status}`);
  }
  
  // Parse and validate with Zod
  const data = response.data;
  return data.web?.results?.map((result: any) => BraveSearchResultSchema.parse(result)) || [];
}

// Task 5: Research Agent with Email Agent as Tool
@researchAgent.tool
async function createEmailDraft(
  ctx: RunContext<AgentDependencies>,
  recipient: string,
  subject: string,
  context: string
): Promise<string> {
  // CRITICAL: Pass usage for token tracking
  const result = await emailAgent.run(
    `Create an email to ${recipient} about: ${context}`,
    { deps: { subject }, usage: ctx.usage }  // PATTERN from multi-agent docs
  );
  
  return `Draft created with ID: ${result.data}`;
}
```

### Integration Points
```yaml
ENVIRONMENT:
  - add to: .env
  - vars: |
      # LLM Configuration
      LLM_PROVIDER=openai
      LLM_API_KEY=sk-...
      LLM_MODEL=gpt-4
      
      # Brave Search
      BRAVE_API_KEY=BSA...
      
      # Gmail (path to credentials.json)
      GMAIL_CREDENTIALS_PATH=./credentials/credentials.json
      
CONFIG:
  - Gmail OAuth: First run opens browser for authorization
  - Token storage: ./credentials/token.json (auto-created)
  
DEPENDENCIES:
  - Update package.json with:
    - googleapis
    - google-auth-library
```

## Validation Loop

### Level 1: Syntax & Style
```bash
# Run these FIRST - fix any errors before proceeding
eslint . --fix              # Auto-fix style issues
tsc --noEmit                # Type checking

# Expected: No errors. If errors, READ and fix.
```

### Level 2: Unit Tests
```typescript
// test_research_agent.test.ts
import { test, describe } from 'node:test';
import assert from 'node:assert';

describe('Research Agent', () => {
  test('searches correctly with Brave API', async () => {
    const agent = createResearchAgent();
    const result = await agent.run("AI safety research");
    assert(result.data);
    assert(result.data.length > 0);
  });

  test('can invoke email agent', async () => {
    const agent = createResearchAgent();
    const result = await agent.run(
      "Research AI safety and draft email to john@example.com"
    );
    assert(result.data.hasOwnProperty('draftId'));
  });
});

// test_email_agent.test.ts  
describe('Email Agent', () => {
  test('handles Gmail OAuth flow', () => {
    process.env.GMAIL_CREDENTIALS_PATH = "test_creds.json";
    const tool = new GmailTool();
    assert(tool.service);
  });

  test('creates draft with proper encoding', async () => {
    const agent = createEmailAgent();
    const result = await agent.run(
      "Create email to test@example.com about AI research"
    );
    assert(result.data.hasOwnProperty('draftId'));
  });
});
```

```bash
# Run tests iteratively until passing:
pnpm test -- --coverage

# If failing: Debug specific test, fix code, re-run
```

### Level 3: Integration Test
```bash
# Test CLI interaction
pnpm run cli

# Expected interaction:
# You: Research latest AI safety developments
# 🤖 Assistant: [Streams research results]
# 🛠 Tools Used:
#   1. brave_search (query='AI safety developments', limit=10)
#
# You: Create an email draft about this to john@example.com  
# 🤖 Assistant: [Creates draft]
# 🛠 Tools Used:
#   1. create_email_draft (recipient='john@example.com', ...)

# Check Gmail drafts folder for created draft
```

## Final Validation Checklist
- [ ] All tests pass: `pnpm test`
- [ ] No linting errors: `eslint .`
- [ ] No type errors: `tsc --noEmit`
- [ ] Gmail OAuth flow works (browser opens, token saved)
- [ ] Brave Search returns results
- [ ] Research Agent invokes Email Agent successfully
- [ ] CLI streams responses with tool visibility
- [ ] Error cases handled gracefully
- [ ] README includes clear setup instructions
- [ ] .env.example has all required variables

---

## Anti-Patterns to Avoid
- ❌ Don't hardcode API keys - use environment variables
- ❌ Don't use sync functions in async agent context
- ❌ Don't skip OAuth flow setup for Gmail
- ❌ Don't ignore rate limits for APIs
- ❌ Don't forget to pass ctx.usage in multi-agent calls
- ❌ Don't commit credentials.json or token.json files

## Confidence Score: 9/10

High confidence due to:
- Clear examples to follow from the codebase
- Well-documented external APIs
- Established patterns for multi-agent systems
- Comprehensive validation gates

Minor uncertainty on Gmail OAuth first-time setup UX, but documentation provides clear guidance.