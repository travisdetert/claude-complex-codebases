# 02 — Setup & Configuration

Configure Claude Code for your project with persistent context, permission settings, and team workflows.

---

## What You'll Learn

- How to create and maintain an effective CLAUDE.md file
- Template CLAUDE.md files for common tech stacks
- How permission modes work and when to adjust them
- The `.claude/` directory and what it contains
- Setting up MCP servers for enhanced capabilities
- How teams share and collaborate on project configuration

**Prerequisites**: [01 — Getting Started](01-getting-started.md)

---

## CLAUDE.md Deep Dive

`CLAUDE.md` is a special file that Claude reads automatically at the start of every session. Place it at the root of your repository and treat it as a living document.

### What to Put in CLAUDE.md

Your CLAUDE.md should contain information that Claude needs in *every* session:

```markdown
# Project Context

## Build & Run
- Install dependencies: `npm install`
- Run the dev server: `npm run dev`
- Run tests: `npm test`
- Run a single test file: `npm test -- path/to/test`
- Lint: `npm run lint`

## Key Architecture
- Express API server in `src/api/`
- React frontend in `src/client/`
- PostgreSQL database, migrations in `src/db/migrations/`
- Background jobs in `src/workers/`

## Conventions
- Use named exports, not default exports
- Error handling: throw AppError instances, not raw Error
- Tests live next to source files: `foo.ts` → `foo.test.ts`
- Database queries go through the repository pattern in `src/db/repos/`

## Gotchas
- The `user` table has a soft-delete (`deleted_at`) — always filter on it
- Config is loaded from environment variables via `src/config.ts`, NOT .env files
- The `legacy/` directory is being migrated — don't add new code there
```

### What NOT to Put in CLAUDE.md

- **Entire file contents** — Claude can read files on its own
- **Information that changes every session** — keep it stable
- **Obvious things** — Claude can figure out that a Python project uses pip
- **Personal preferences** — use Claude's memory system for those (see below)
- **Secrets or API keys** — CLAUDE.md is typically committed to the repo

### When to Update It

Update your CLAUDE.md when you discover:
- A build/test command that isn't obvious
- A convention that Claude keeps getting wrong
- An architectural decision that requires context
- A "gotcha" that wastes time if you don't know about it

A good rule of thumb: if you find yourself repeating the same instruction to Claude, put it in CLAUDE.md.

---

## Template CLAUDE.md Files

### Node.js / Express

```markdown
# Project Context

## Build & Run
- Install: `npm install`
- Dev server: `npm run dev` (uses nodemon)
- Tests: `npm test` (Jest)
- Single test: `npm test -- --testPathPattern=<pattern>`
- Lint: `npm run lint` (ESLint + Prettier)
- Build: `npm run build` (TypeScript → dist/)

## Architecture
- Express API in `src/routes/`
- Middleware in `src/middleware/`
- Business logic in `src/services/`
- Database access in `src/repositories/`
- TypeScript throughout, strict mode enabled

## Conventions
- Use async/await, not callbacks
- Validate request input with Zod schemas in `src/schemas/`
- All API responses follow the shape: `{ data, error, meta }`
- Environment config via `src/config.ts` using envalid
```

### Python / Django

```markdown
# Project Context

## Build & Run
- Install: `pip install -r requirements.txt`
- Dev server: `python manage.py runserver`
- Tests: `pytest` (not manage.py test)
- Single test: `pytest path/to/test.py::TestClass::test_method`
- Lint: `ruff check .`
- Migrations: `python manage.py migrate`
- Create migration: `python manage.py makemigrations`

## Architecture
- Django apps in `apps/` directory
- API views use Django REST Framework
- Celery tasks in each app's `tasks.py`
- Settings split: `config/settings/{base,dev,prod}.py`

## Conventions
- Use class-based views for CRUD, function views for custom endpoints
- All model changes need a migration
- Use `get_object_or_404`, not manual try/except on queries
- Serializers handle both validation and output formatting
```

### React SPA

```markdown
# Project Context

## Build & Run
- Install: `npm install`
- Dev server: `npm run dev` (Vite)
- Tests: `npm test` (Vitest + React Testing Library)
- Build: `npm run build`
- Storybook: `npm run storybook`

## Architecture
- Pages in `src/pages/` (maps to routes)
- Shared components in `src/components/`
- API calls in `src/api/` (uses TanStack Query)
- State management: Zustand stores in `src/stores/`
- Styling: Tailwind CSS with component classes

## Conventions
- Components are function components with TypeScript
- Co-locate tests: `Button.tsx` → `Button.test.tsx`
- Use `cn()` utility for conditional class merging
- API hooks wrap TanStack Query: `useUsers()`, `useCreateUser()`
```

### Java / Spring Boot

```markdown
# Project Context

## Build & Run
- Build: `./gradlew build`
- Run: `./gradlew bootRun`
- Tests: `./gradlew test`
- Single test: `./gradlew test --tests "com.example.MyTest"`
- Lint: `./gradlew spotlessCheck`
- Format: `./gradlew spotlessApply`

## Architecture
- Controllers in `src/main/java/.../controller/`
- Services in `src/main/java/.../service/`
- Repositories (Spring Data JPA) in `src/main/java/.../repository/`
- DTOs in `src/main/java/.../dto/`
- Configuration in `src/main/resources/application.yml`

## Conventions
- Use constructor injection, not @Autowired on fields
- DTOs for API input/output, entities for persistence
- Exceptions extend AppException and are handled by GlobalExceptionHandler
- Use MapStruct for entity ↔ DTO mapping
```

### Monorepo

```markdown
# Project Context

## Structure
This is a monorepo managed by [Turborepo/Nx/Lerna].

- `packages/shared/` — shared types and utilities
- `packages/api/` — Express API server
- `packages/web/` — React frontend
- `packages/worker/` — background job processor

## Build & Run
- Install all: `npm install` (workspaces)
- Dev (all): `npm run dev`
- Dev (single): `npm run dev --workspace=packages/api`
- Tests (all): `npm test`
- Tests (single): `npm test --workspace=packages/web`

## Conventions
- Shared types go in `packages/shared/src/types/`
- Cross-package imports use `@myorg/shared` (not relative paths)
- Each package has its own tsconfig extending the root
- Database migrations are in `packages/api/migrations/`
```

---

## Permission Modes

Claude Code has permission modes that control how much autonomy Claude has:

### How Permissions Work

- **Read-only actions** (reading files, searching code) are auto-approved
- **Write actions** (editing files, running commands) prompt for confirmation
- **Destructive actions** (deleting files, force-pushing) always prompt

Use `/permissions` inside a session to review your current settings.

### Adjusting for Your Workflow

When **exploring** an unfamiliar codebase (Guides 03–05), the defaults are perfect — you want Claude to read freely but ask before changing anything.

When **executing tasks** (Guide 06), you may want to allow specific commands (like `npm test`) to auto-approve so Claude can iterate faster.

---

## The `.claude/` Directory

Claude Code creates a `.claude/` directory in your project for local configuration:

```
.claude/
├── settings.json     # Project-specific settings
└── memory/           # Claude's memory files for this project
```

### settings.json

Stores project-specific permission settings, allowed/denied commands, and other configuration. This is managed through the `/permissions` command.

### Memory Files

Claude can store notes in `.claude/memory/` that persist across sessions. This is different from CLAUDE.md:

| | CLAUDE.md | `.claude/memory/` |
|--|-----------|-------------------|
| **Scope** | Team-wide (committed to repo) | Personal (gitignored) |
| **Content** | Project conventions, build commands | Personal preferences, notes |
| **Updated by** | You or Claude on request | Claude automatically or on request |

---

## MCP Servers

Model Context Protocol (MCP) servers extend Claude's capabilities with specialized tools. Common setups:

### GitHub Integration

Gives Claude direct access to issues, PRs, and repository metadata:

```bash
claude mcp add github -- npx -y @modelcontextprotocol/server-github
```

### Database Access

Connect Claude to your development database for schema exploration:

```bash
claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres $DATABASE_URL
```

### File System (Extended)

Grant Claude access to directories outside the project root:

```bash
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem /path/to/other/directory
```

Only add MCP servers you actively need. Each server adds capabilities but also increases what Claude can access.

---

## Team Workflows

### Sharing CLAUDE.md

Commit your `CLAUDE.md` to the repository. It benefits everyone:

1. New team members get project context immediately
2. Claude behaves consistently across the team
3. Conventions are documented and enforced naturally

### Multiple CLAUDE.md Files

You can place `CLAUDE.md` files at different levels:

```
repo-root/
├── CLAUDE.md              # Repo-wide context
├── packages/
│   ├── api/
│   │   └── CLAUDE.md      # API-specific context
│   └── web/
│       └── CLAUDE.md      # Frontend-specific context
```

Claude reads all `CLAUDE.md` files from the root down to the current directory. More specific files supplement (not replace) broader ones.

### What to Gitignore

Add `.claude/` to your `.gitignore` — it contains personal settings and memory that shouldn't be shared:

```gitignore
.claude/
```

---

## Key Takeaways

1. `CLAUDE.md` is your most important configuration — invest time in it and keep it updated
2. Put commands, conventions, and gotchas in CLAUDE.md; leave obvious things out
3. Permission defaults are safe; adjust only when you have a specific reason
4. Use `.claude/memory/` for personal notes, `CLAUDE.md` for team knowledge
5. Commit `CLAUDE.md` to your repo; gitignore `.claude/`

---

**Next**: [03 — Codebase Orientation](03-codebase-orientation.md) — Build a mental map of any codebase.
