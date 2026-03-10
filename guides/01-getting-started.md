# 01 — Getting Started with Claude Code

Get Claude Code installed, launch your first session, and understand the core concepts you'll use in every interaction.

---

## What You'll Learn

- What Claude Code is and how it differs from other AI coding tools
- How to install and launch Claude Code
- Core concepts: context window, tools, sessions
- How to read Claude's output (tool calls, permission prompts, diffs)
- A hands-on 5-minute exercise you can follow on any project

**Prerequisites**: None — this is the starting point.

---

## What Is Claude Code?

Claude Code is a command-line tool that brings Claude directly into your terminal. It can read your files, run commands, edit code, and interact with git — all through natural language conversation.

Think of it as a senior engineer sitting next to you who can instantly read and understand any file in your project.

### How It Compares to Other Tools

| Feature | Claude Code | Chat-based AI | IDE Extensions |
|---------|------------|---------------|----------------|
| Reads your files directly | Yes | No (paste manually) | Partial |
| Runs shell commands | Yes | No | Some |
| Edits files in place | Yes | No | Yes |
| Works with any language/framework | Yes | Yes | Varies |
| Git integration | Native | No | Some |
| Works outside an IDE | Yes | Yes | No |
| Persistent project context | Yes (CLAUDE.md) | No | Varies |

The key difference: Claude Code operates *in* your project directory with full filesystem access. You don't paste code snippets — Claude reads and writes real files.

---

## Installation

### Prerequisites

- Node.js 18+ (check with `node --version`)
- An Anthropic API key or Claude Pro/Max subscription

### Install via npm

```bash
npm install -g @anthropic-ai/claude-code
```

### Verify the Installation

```bash
claude --version
```

---

## First Launch

Navigate to the root of any project and start Claude:

```bash
cd /path/to/your/project
claude
```

That's it. Claude now has access to the files in this directory and below. You talk to it by typing natural language — no special syntax required.

On first launch, Claude will ask you to authenticate. Follow the prompts to connect your Anthropic account.

---

## Key Concepts

### Context Window

Claude can hold a limited amount of information in a single conversation. It manages this automatically, but it has practical implications:

- **Focus conversations on specific areas** rather than asking about the entire codebase in one prompt
- Use `/compact` to compress earlier conversation when the context gets full
- Start new sessions (`/clear` or exit and re-run `claude`) for unrelated tasks

Think of it like a whiteboard — eventually you run out of space and need to erase older notes to keep working.

### Tools

Claude interacts with your system through tools — structured actions like reading files, searching code, running commands, and editing files. When Claude uses a tool, you'll see it in the output:

```
Claude wants to run: cat package.json

Allow? (y/n)
```

Read-only actions (reading files, searching) are typically auto-approved. Actions that modify your system (running commands, editing files) prompt for confirmation unless you've configured auto-approval.

### Sessions

Each time you run `claude`, you start a fresh session. Claude doesn't remember previous conversations unless you:

1. Create a `CLAUDE.md` file with persistent context (covered in [Guide 02](02-setup-and-configuration.md))
2. Use Claude's memory system for personal preferences

### The `/compact` Command

As your conversation grows, it fills the context window. The `/compact` command compresses the conversation history into a summary, freeing up space while preserving the key points. Use it when:

- Claude mentions the context is getting long
- You're shifting to a different topic within the same session
- You notice Claude forgetting earlier context

---

## Understanding Claude's Output

When Claude works, you'll see several types of output:

### Tool Calls

Claude shows what tool it's using and why:

```
I'll read the package.json to understand the project dependencies.

📖 Read file: package.json
```

### Permission Prompts

When Claude wants to do something potentially impactful, it asks first:

```
Claude wants to execute: npm test

Allow? (y/n/always)
```

Choosing "always" auto-approves that specific command pattern for the rest of the session.

### Diffs

When Claude edits a file, it shows you exactly what changed:

```diff
- const MAX_RETRIES = 3;
+ const MAX_RETRIES = 5;
```

Always review diffs before accepting — especially in unfamiliar code.

---

## Your First 5 Minutes: A Guided Exploration

Try this exercise on any project. It takes about 5 minutes and gives you a feel for the workflow.

### Step 1: Launch Claude in a Project

```bash
cd /path/to/any/project
claude
```

### Step 2: Ask for the Big Picture

Type this prompt:

```
Give me a quick overview of this project. What does it do,
what's the tech stack, and how is the code organized?
```

Watch how Claude reads the directory structure, config files, and key source files to answer.

### Step 3: Ask a Follow-Up

Based on Claude's response, ask something specific:

```
What's the main entry point? Walk me through what happens
when the application starts.
```

### Step 4: Look at Something Specific

Pick any file that caught your eye from Claude's overview:

```
Explain what [filename] does. What's the most important
function in this file?
```

### Step 5: Ask What to Explore Next

```
If I were a new developer on this project, what are the
three most important things to understand after this overview?
```

That's it. In 5 minutes, you've gone from zero to a working mental model of the project. The rest of this guide series builds on this foundation.

---

## Quick Reference

| Command / Action | What It Does |
|------------------|--------------|
| `claude` | Start a new session in the current directory |
| `/help` | Show available commands |
| `/clear` | Clear conversation history and start fresh |
| `/compact` | Compress conversation to free up context |
| `Escape` | Cancel current input |
| `Ctrl+C` | Interrupt Claude while it's working |

---

## Key Takeaways

1. Claude Code works *inside* your project — it reads and writes real files, not pasted snippets
2. Focus conversations on specific areas; use `/compact` when the context gets full
3. Claude asks permission before doing anything destructive — read the prompts
4. Always review diffs before accepting edits
5. Start with a broad overview, then drill into specifics — the 5-minute exercise pattern works on any project

---

**Next**: [02 — Setup & Configuration](02-setup-and-configuration.md) — Set up persistent context and configure Claude for your project.
