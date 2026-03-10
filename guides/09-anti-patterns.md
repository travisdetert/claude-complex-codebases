# 09 — Anti-Patterns

Common mistakes when using Claude Code — why people make them, what goes wrong, and what to do instead.

---

## What You'll Learn

- 11 anti-patterns that waste time, introduce bugs, or produce poor results
- Why each pattern is tempting (it's not obvious that it's wrong)
- What actually goes wrong when you do it
- What to do instead, with example prompts

**Prerequisites**: None — read this anytime. It's most useful after you've used Claude Code a few times and want to improve your workflow.

---

## 1. Skipping Exploration

### The Pattern
Jumping straight to "fix bug X" or "add feature Y" on an unfamiliar codebase without understanding the system first.

### Why People Do It
It feels productive. You have a task, you want to start. Exploration feels like procrastination.

### What Goes Wrong
- You miss architectural patterns and create inconsistent code
- You change the wrong layer (fixing the symptom, not the cause)
- You break things you didn't know existed
- You duplicate functionality that already exists somewhere else

### What to Do Instead

Spend 15–30 minutes in orientation first (see [Guide 03](03-codebase-orientation.md)):

```
Before I work on anything, give me a quick overview of this
codebase — architecture, key patterns, and how the area
around [my task] is structured.
```

The time invested pays back immediately by preventing wrong-direction work.

---

## 2. Making Sweeping Changes Without Understanding Tests

### The Pattern
Refactoring a module, updating an API, or changing core logic without knowing what test coverage exists.

### Why People Do It
Claude makes refactoring easy, so it's tempting to "clean up" code along the way. The tests will catch problems, right?

### What Goes Wrong
- If coverage is thin, the tests DON'T catch problems — you ship bugs
- If tests exist but you don't understand them, you might "fix" the tests instead of fixing your code
- You can't distinguish between "test catches a real regression" and "test needs updating for new behavior"

### What to Do Instead

Read the tests first. Assess coverage. Add tests if needed:

```
Show me all tests related to [module]. What behaviors do they
verify? Where are the gaps?
```

```
Before we refactor this, write tests that capture the current
behavior. I want a safety net.
```

---

## 3. Taking Claude's First Answer as Complete

### The Pattern
Accepting Claude's initial architecture summary or code explanation without follow-up questions.

### Why People Do It
The first answer is usually coherent and detailed — it feels complete. Asking follow-ups feels like you're doubting Claude.

### What Goes Wrong
- The first pass typically covers 70–80% of the picture
- Important nuances, edge cases, and non-obvious patterns get missed
- You build a mental model with gaps — those gaps bite you later

### What to Do Instead

Always follow up:

```
What did you miss in that summary? Are there non-obvious
aspects of this architecture that weren't covered?
```

```
What about error handling? You didn't mention how errors
flow through the system.
```

```
Are there any parts of the codebase that contradict or
don't fit the patterns you described?
```

---

## 4. Ignoring the Build Pipeline

### The Pattern
Writing code without understanding how it ships — CI/CD, linting rules, deployment requirements.

### Why People Do It
The pipeline is "DevOps stuff" that seems separate from the code. You'll deal with it when you push.

### What Goes Wrong
- Your code passes locally but fails CI because of linting rules you didn't know about
- You add a dependency that the CI environment doesn't have
- You change a config format that breaks deployment
- You waste cycles fixing pipeline failures after the fact

### What to Do Instead

Understand the pipeline early (see [Guide 04](04-architecture-and-dependencies.md)):

```
Walk me through the CI/CD pipeline. What checks run on PRs?
What linting and formatting rules are enforced? What would
cause my PR to fail?
```

---

## 5. Changing Code You Don't Understand

### The Pattern
Letting Claude rewrite something because it looks messy, overly complex, or "not how I'd do it."

### Why People Do It
Clean code is satisfying. Claude makes refactoring easy. The old code looks like it needs help.

### What Goes Wrong
- "Messy" code often handles real edge cases that clean code ignores
- Workarounds exist for real bugs in dependencies or infrastructure
- You introduce regressions for problems you don't know exist yet
- The original author knew something you don't

### What to Do Instead

Explain before change (see [Guide 08](08-ongoing-practices.md)):

```
Explain what this code does and WHY it does it this way.
Is there a reason for this complexity that isn't obvious?
Check the git history for context.
```

Only after understanding the intent should you consider simplifying.

---

## 6. Losing Context Between Sessions

### The Pattern
Starting every Claude session from scratch, re-explaining the project and your goals.

### Why People Do It
Sessions end, memory fades. Without persistent context, each session starts cold.

### What Goes Wrong
- You waste 5–10 minutes per session re-establishing context
- Claude may give different or inconsistent analysis of the same code
- Knowledge you gathered in previous sessions is lost
- You never build momentum

### What to Do Instead

Maintain CLAUDE.md and update it at the end of each session (see [Guide 02](02-setup-and-configuration.md)):

```
Update CLAUDE.md with the key things we learned this session
— conventions, gotchas, architecture notes, and anything that
would help future sessions.
```

---

## 7. The Shotgun Prompt

### The Pattern
Asking for too many things in a single prompt:

```
Refactor the auth module, add rate limiting, update the tests,
fix the bug in the login flow, and add TypeScript types to the
user service.
```

### Why People Do It
It feels efficient. You know everything you need done, so why not ask for it all at once?

### What Goes Wrong
- Claude may conflate the tasks, making changes for one that break another
- It's hard to review the diff when five unrelated changes are interleaved
- If something goes wrong, you can't tell which change caused it
- The context window fills up with too many concerns at once

### What to Do Instead

One task at a time. Commit between tasks:

```
Let's start with the auth module refactor. We'll handle
rate limiting after that.
```

If tasks are truly independent, use subagents instead:

```
Research these three areas in parallel and give me a summary
of each — but don't make any changes yet.
```

---

## 8. The Copy-Paste Loop

### The Pattern
When something fails, copying the error message and pasting it back to Claude without any context:

```
Fix this:
Error: Cannot read property 'map' of undefined
    at UserList.render (src/components/UserList.tsx:42:18)
```

### Why People Do It
It's the fastest thing to do. The error message should contain enough information.

### What Goes Wrong
- Claude may "fix" the symptom (add a null check) without fixing the root cause (data not loading)
- Without context about what you were trying to do, Claude can't prioritize approaches
- You end up in a loop: fix symptom → hit next error → fix that symptom → repeat

### What to Do Instead

Provide context with the error:

```
I'm trying to [what you were doing] and got this error:
[error message]

Before fixing it, explain why this is happening. What's
the root cause? Is this a symptom of a deeper issue?
```

---

## 9. The Trust Fall

### The Pattern
Accepting large refactors or multi-file changes from Claude without carefully reviewing the diff.

### Why People Do It
Claude's explanations are convincing. The code looks clean. The tests pass. Why review 400 lines of changes?

### What Goes Wrong
- Subtle behavior changes that tests don't cover
- Removed error handling that seemed redundant but wasn't
- Changed public APIs that break downstream callers
- Introduced patterns inconsistent with the rest of the codebase

### What to Do Instead

Always review the diff, especially for large changes:

```
Show me a summary of every file we changed and what changed
in each. Highlight anything that changes behavior (not just
code style).
```

For large refactors, break them into smaller, reviewable steps:

```
Let's do this refactor incrementally. What's the smallest
first step we can make and verify?
```

---

## 10. The Infinite Context

### The Pattern
Never using `/compact` and running into context window limits mid-task.

### Why People Do It
They don't think about context limits until Claude starts forgetting things or the session gets slow.

### What Goes Wrong
- Claude starts losing track of earlier decisions and context
- Responses get slower as the context fills up
- You may hit hard limits and lose the ability to continue
- Claude may contradict earlier analysis without realizing it

### What to Do Instead

Use `/compact` proactively:

- After finishing exploration and before starting implementation
- After a complex discussion when you've reached a decision
- Whenever you shift topics within the same session
- As a habit every 20–30 minutes of active conversation

```
/compact
```

Then re-state your current focus:

```
We're implementing [feature]. The plan is [brief summary].
Next step is [what you're about to do].
```

---

## 11. The Solo Explorer

### The Pattern
Exploring multiple areas of a large codebase sequentially, one at a time, in a single conversation.

### Why People Do It
It's the natural way — look at one thing, then the next, then the next.

### What Goes Wrong
- Each exploration fills the context window, crowding out earlier findings
- It takes 3x longer than it needs to
- By the time you finish exploring area 3, you've forgotten the details of area 1

### What to Do Instead

Use subagents for parallel exploration:

```
Research these areas in parallel:
1. How the authentication system works — key files, flow, patterns
2. How the payment processing works — key files, flow, patterns
3. How the notification system works — key files, flow, patterns

Give me a structured summary of each.
```

This runs the explorations concurrently and gives you a clean summary of all three without context window pollution.

---

## Quick Reference

| Anti-Pattern | One-Line Fix |
|-------------|-------------|
| Skipping exploration | Spend 15 min on orientation first |
| No test awareness | Read tests before changing code |
| First answer = final answer | Always follow up with probing questions |
| Ignoring build pipeline | Learn CI/CD early |
| Changing mystery code | Explain before change |
| Context amnesia | Maintain CLAUDE.md |
| Shotgun prompt | One task at a time |
| Copy-paste loop | Add context with errors |
| Trust fall | Review every diff |
| Infinite context | `/compact` proactively |
| Solo explorer | Use subagents for parallel work |

---

## Key Takeaways

1. Most anti-patterns stem from impatience — taking 5 minutes upfront saves 30 minutes of rework
2. Claude's first answer is a starting point, not a conclusion — always follow up
3. One task per prompt, one commit per task — keep changes atomic and reviewable
4. Context management is your responsibility — use `/compact` before you need it
5. The diff is the truth — no matter how good the explanation sounds, review what actually changed

---

**Next**: [10 — Migration Planning](10-migration-planning.md) — Plan and execute large-scale migrations safely.
