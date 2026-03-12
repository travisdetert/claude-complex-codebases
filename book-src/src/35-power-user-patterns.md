# 35 — Power User Patterns

Advanced techniques for experienced Claude Code users — context management, custom tooling, session strategies, and workflow optimization.

---

## What You'll Learn

- Context window management — keeping Claude effective in long sessions
- Session strategies for different types of work
- Custom MCP servers for your specific workflows
- Advanced prompting techniques that compound over time
- Chaining tasks across sessions
- Building internal dev tools with Claude
- Extending Claude Code with hooks and custom commands
- Workflow patterns used by power users

**Prerequisites**: [27 — Rules & Instructions](27-rules-and-instructions.md), [25 — MCP Servers](25-mcp-servers.md), [26 — AI Agents & Agentic Patterns](26-ai-agents-and-agentic-patterns.md)

---

## Context Window Management

The context window is your most valuable resource. Treat it like RAM — when it fills up, performance degrades.

### How Context Gets Used

![Diagram](images/mermaid_cfe44aab24c8e1b5cb404bc0d533fbd3.png)

As your conversation grows, the context fills with:
- Every file Claude has read
- Every tool result
- Every exchange between you and Claude
- CLAUDE.md rules (loaded every session)

When context gets full, Claude's performance degrades — it loses track of earlier context, makes mistakes, and starts repeating itself.

### The Compact Command

When you notice degradation, compact the context:

```
/compact
```

Then re-state your current goal clearly:

```
We're adding OAuth2 to the auth system. We've finished
the token refresh logic. Next step: implement the
authorization callback endpoint in src/api/auth/.
```

### When to Compact

- After reading many large files
- When Claude starts repeating itself or losing track
- When switching from exploration to implementation
- Every 15-20 exchanges in a complex task
- Before starting a new phase of work in the same session

### When to Start Fresh Instead

Sometimes `/compact` isn't enough — start a new session when:

- You've changed direction significantly
- Claude is confused despite compacting
- You want a fresh perspective (second opinion pattern)
- The session has accumulated too many failed attempts

Carry forward what you learned:

```
I'm continuing work on the OAuth2 integration.

Completed:
- Token refresh logic in src/services/auth/tokenService.ts
- Token storage in Redis via src/cache/tokenCache.ts

Current task:
- Implement the authorization callback endpoint
- File: src/api/controllers/authController.ts
- Follow the pattern of existing endpoints in that file

Constraints:
- Use our standard ApiResponse<T> wrapper
- Must work with both Google and GitHub providers
```

---

## Session Strategies

Different types of work call for different session strategies.

### The Exploration Session

**Goal**: Understand a codebase or investigate a problem. Don't make changes.

```
I'm exploring, not implementing. Do NOT make any changes.
Just read code and explain what you find.

Start by mapping the authentication flow from login
request to session creation. Show me every file involved.
```

**Why separate**: Exploration reads many files, filling context. If you then try to implement in the same session, Claude has less room for the implementation work.

### The Focused Implementation Session

**Goal**: Make one specific change. Get in, make the change, get out.

```
Single task: add rate limiting to POST /api/orders.
10 requests per minute per user. Use our existing
rateLimit middleware pattern from src/middleware/rateLimit.ts.
Return 429 with Retry-After header. Write tests.
```

**Why focused**: Narrow scope = better results. Claude doesn't waste context on unrelated exploration.

### The Multi-Phase Session

**Goal**: A larger task broken into phases. Compact between phases.

```
Phase 1: Analyze the current notification system and
create a plan for adding WebSocket support.
Don't implement anything yet.
```

*Review the plan, then:*

```
/compact
```

```
Phase 2: We're adding WebSocket support to notifications.
Plan summary: [paste the key points from Phase 1].
Implement the WebSocket server in src/services/websocket/.
```

### The Code Review Session

**Goal**: Review a PR or a set of changes. Read-only analysis.

```
Review this PR. Don't make changes — just analyze.

Focus on:
1. Correctness (does the logic actually work?)
2. Security (any vulnerabilities?)
3. Performance (any N+1 queries or unbounded operations?)
4. Missing tests
5. Deviations from our codebase patterns

Be specific — reference files and line numbers.
```

---

## Advanced Prompting Techniques

### The Constraint Stack

Layer constraints to get precisely what you want:

```
Add pagination to the /api/products endpoint.

Constraints:
- Use cursor-based pagination, not offset-based
- Follow the existing pagination pattern in src/api/controllers/orderController.ts
- Max page size: 100 items
- Default page size: 20 items
- Response format must match our standard: { data, meta: { cursor, hasMore } }
- Don't modify the database query — use the existing ProductRepository methods
- Write tests covering: first page, middle page, last page, empty result, invalid cursor
```

More constraints = more predictable output. Spend time writing good constraints instead of fixing bad output.

### The Breadcrumb Pattern

For complex tasks, leave breadcrumbs that help Claude maintain context:

```
We're implementing a 3-step checkout flow:
  Step 1: Cart validation ← WE ARE HERE
  Step 2: Payment processing
  Step 3: Order confirmation

For Step 1, validate that:
- All items in the cart are in stock
- Prices haven't changed since the cart was created
- The user's shipping address is complete
```

After completing Step 1:

```
We're implementing a 3-step checkout flow:
  Step 1: Cart validation ✓ DONE
  Step 2: Payment processing ← WE ARE HERE
  Step 3: Order confirmation

Step 1 created: CartValidator in src/services/checkout/cartValidator.ts
Now implement Step 2...
```

### The Example-Driven Pattern

Show Claude what you want by pointing to existing code:

```
Create a new API endpoint for /api/invoices.

The exact pattern to follow is in src/api/controllers/orderController.ts:
- Same file structure
- Same error handling pattern
- Same response wrapper
- Same middleware chain (auth, validation, rate limit)
- Same test structure in __tests__/

Read that file first, then replicate the pattern for invoices.
```

### The Negative Constraint

Tell Claude what NOT to do — this prevents common failure modes:

```
Add caching to the product catalog API.

DO NOT:
- Change the database queries
- Modify the ProductRepository
- Add new dependencies
- Change the response format
- Add cache invalidation yet (we'll do that next)

ONLY:
- Add a Redis cache layer in front of the existing repository calls
- Cache for 5 minutes
- Use our existing Redis client from src/cache/client.ts
```

---

## Custom MCP Servers for Your Workflow

### Internal Tools Server

Build an MCP server that connects Claude to your internal tools:

```javascript
// tools/internal-mcp-server.js
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "internal-tools",
  version: "1.0.0",
});

// Check feature flag status
server.tool(
  "check_feature_flag",
  "Check if a feature flag is enabled in a given environment",
  {
    flag: z.string().describe("Feature flag name"),
    environment: z.enum(["dev", "staging", "production"]),
  },
  async ({ flag, environment }) => {
    const response = await fetch(
      `${FEATURE_FLAG_API}/flags/${flag}?env=${environment}`,
      { headers: { Authorization: `Bearer ${process.env.FF_TOKEN}` } }
    );
    const data = await response.json();
    return {
      content: [{ type: "text", text: JSON.stringify(data, null, 2) }],
    };
  }
);

// Look up on-call schedule
server.tool(
  "get_oncall",
  "Get the current on-call engineer for a service",
  {
    service: z.string().describe("Service name"),
  },
  async ({ service }) => {
    const response = await fetch(
      `${PAGERDUTY_API}/oncalls?service=${service}`,
      { headers: { Authorization: `Token ${process.env.PD_TOKEN}` } }
    );
    const data = await response.json();
    return {
      content: [{ type: "text", text: JSON.stringify(data, null, 2) }],
    };
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

Configure in `.claude/settings.json`:

```json
{
  "mcpServers": {
    "internal": {
      "command": "node",
      "args": ["tools/internal-mcp-server.js"],
      "env": {
        "FF_TOKEN": "${FEATURE_FLAG_TOKEN}",
        "PD_TOKEN": "${PAGERDUTY_TOKEN}"
      }
    }
  }
}
```

Now Claude can check feature flags and on-call schedules during conversations:

```
Is the "new-checkout" flag enabled in staging?
Who's on call for the payment service right now?
```

### Database Query Server

A read-only database MCP server for your development database:

```json
{
  "mcpServers": {
    "dev-db": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://readonly:password@localhost:5432/myapp_dev"
      }
    }
  }
}
```

**Important**: Use a read-only database user. Never give Claude write access to a production database through MCP.

---

## Extending Claude Code with Hooks

Hooks let you run shell commands automatically before or after Claude uses specific tools.

### Auto-Format on Save

```json
{
  "hooks": {
    "postToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "npx prettier --write $CLAUDE_FILE_PATH"
      }
    ]
  }
}
```

Every time Claude writes or edits a file, Prettier formats it automatically.

### Auto-Lint Check

```json
{
  "hooks": {
    "postToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "npx eslint --fix $CLAUDE_FILE_PATH 2>/dev/null; true"
      }
    ]
  }
}
```

### Custom Validation

```json
{
  "hooks": {
    "postToolUse": [
      {
        "matcher": "Write",
        "command": "scripts/validate-new-file.sh $CLAUDE_FILE_PATH"
      }
    ]
  }
}
```

Where `validate-new-file.sh` checks naming conventions, required headers, etc.

---

## Workflow Patterns

### The Morning Standup

Start each work session with a quick context load:

```
I'm starting work on the checkout feature. Here's where I left off:

Completed:
- Cart API endpoints (PR #234, merged)
- Cart UI components (PR #237, in review)

Today's plan:
- Implement payment processing endpoint
- Write integration tests for Stripe webhook handling

Any recent changes to src/api/ or src/services/payment/
I should know about?
```

### The Rubber Duck Session

Use Claude as a thinking partner, not just a code generator:

```
I'm trying to decide between two approaches for handling
concurrent order processing. Don't write code — just
help me think through the tradeoffs.

Approach A: Optimistic locking with retry
Approach B: Pessimistic locking with queue

We have ~50 orders per minute, and ~5% of orders modify
the same inventory items concurrently.
```

### The Spec-First Workflow

Write a spec before any code:

```
I want to add a notification preferences system.
Before writing any code, create a spec:

1. Data model (what tables/columns)
2. API endpoints (method, path, request/response)
3. Business rules (what can be configured, defaults)
4. Edge cases (what happens when...)

Format as a markdown document I can review with the team.
```

Review and iterate on the spec. Then:

```
Implement the spec we just defined. Here it is for reference:
[paste or point to the spec]
```

### The Incremental Refactor

For large refactors, use an incremental approach:

```
We need to migrate from Moment.js to date-fns across
the entire codebase. Don't do it all at once.

Step 1: Show me all files that import Moment.
Step 2: For each file, assess the complexity of migration.
Step 3: Start with the simplest files first.
Step 4: After each file, run tests to verify.

Do one file at a time. Show me the diff after each one
so I can verify before moving on.
```

### The Knowledge Extraction Session

Use Claude to extract and document tribal knowledge:

```
Analyze the codebase and extract implicit knowledge
that isn't documented anywhere:

1. What naming conventions are followed but not written down?
2. What patterns are used consistently but not documented?
3. What are the unwritten rules about code organization?
4. What gotchas would a new developer hit?

Format this as additions to our CLAUDE.md so future
sessions benefit from this knowledge.
```

---

## Building Internal Dev Tools

### Script Generator

Use Claude to create project-specific scripts:

```
Create a CLI script at scripts/seed-dev-data.sh that:
1. Resets the dev database
2. Runs all migrations
3. Seeds with realistic test data (50 users, 200 orders, etc.)
4. Creates a test admin account (admin@test.com / password)
5. Prints a summary of what was created

Make it idempotent — safe to run multiple times.
```

### Custom Code Generators

```
Create a code generator script at scripts/generate-endpoint.sh
that takes a resource name (e.g., "invoice") and generates:
1. Controller file following our pattern
2. Service file following our pattern
3. Repository file following our pattern
4. Route registration
5. Test file with standard test structure
6. OpenAPI documentation stub

Usage: ./scripts/generate-endpoint.sh invoice
```

### Project Health Dashboard

```
Create a script at scripts/project-health.sh that reports:
1. Test coverage percentage
2. Number of TODO/FIXME/HACK comments
3. Dependencies with known vulnerabilities
4. Files with no test coverage
5. Largest files (potential refactoring candidates)
6. Circular dependencies

Output as a formatted terminal report.
```

---

## Performance Tips

### Reduce CLAUDE.md Size

Every line of CLAUDE.md uses context every session. Audit regularly:

```
Review our CLAUDE.md and identify:
1. Rules that Claude already follows by default (remove them)
2. Rules that are too vague to be actionable (rewrite or remove)
3. Rules that duplicate each other (consolidate)
4. Information that belongs in docs, not instructions (move it)

Keep it under 100 lines of actual rules.
```

### Pre-Load Context Efficiently

Instead of having Claude read 20 files:

```
The relevant code for this task is in these 3 files:
- src/services/payment/processor.ts (the main service)
- src/services/payment/types.ts (the type definitions)
- src/api/controllers/paymentController.ts (the API layer)

Read these three files, then we'll discuss the change.
```

Targeted reads > exploratory reads for focused tasks.

### Use Subagents for Research

When you need research but don't want to pollute your main context:

```
Use a subagent to research how error handling works across
all API controllers. Summarize the patterns found — don't
dump all the file contents into our conversation.
```

The subagent does the heavy reading in its own context. Your main session stays clean.

---

## Sample Conversation

> **You**: I'm a power user and I feel like my sessions are getting less effective as they go longer. What should I change?
>
> **Claude**: Let me look at how your typical session flows...
>
> Based on common patterns, here are the likely issues:
>
> **1. Context bloat from exploration**
> If you're reading many files early in the session (exploring), then switching to implementation, the context is full of exploration results you no longer need.
>
> **Fix**: Separate exploration and implementation into different sessions, or `/compact` between phases.
>
> **2. CLAUDE.md might be too large**
> *reads CLAUDE.md*
>
> Your CLAUDE.md is 180 lines. That's context used every session. I see several rules that are default behavior ("handle errors properly") and some that could be consolidated. Want me to trim it?
>
> **3. Accumulating failed attempts**
> If you've tried 3 approaches that didn't work, those failed attempts are still in context, confusing subsequent attempts.
>
> **Fix**: When an approach fails, start a new session and explicitly say "Don't try [failed approach] because [reason]."
>
> **4. Not using subagents for research**
> When you need to understand something (like "how does auth work across all controllers?"), the research fills your context. Subagents do the research in their own context and return a summary.
>
> Want me to trim your CLAUDE.md now? That alone will improve every session.

---

## Key Takeaways

1. The context window is your most valuable resource — manage it actively with `/compact` and fresh sessions
2. Match your session strategy to the work type — exploration, focused implementation, and multi-phase work need different approaches
3. More constraints = more predictable output — invest time in precise prompts instead of fixing imprecise results
4. Custom MCP servers connect Claude to your specific tools — feature flags, internal APIs, monitoring dashboards
5. Hooks automate the tedious — auto-format, auto-lint, and custom validation on every file change
6. Use the negative constraint pattern ("DO NOT change X") to prevent common failure modes
7. Subagents keep your main context clean — delegate research to a subagent, get back a summary
8. Audit CLAUDE.md regularly — keep it under 100 lines of actionable rules

---

**Next**: Back to the [main page](../README.md) to explore all guides by track.
