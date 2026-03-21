# SpawnDock Workspace

This repository is the development workspace for the **SpawnDock** platform

It combines the source code for all SpawnDock components with a spec-driven development workflow powered by **speckit**

## Repository Layout

```
.
├── repo/                        # Source code (git submodules)
│   ├── api/                     # Backend: Telegram bot + MCP server + tunnel control plane
│   ├── tma-project/             # TMA starter template (Next.js + TON Connect + TelegramUI)
│   ├── mcp-client/              # @spawn-dock/mcp — stdio-to-SSE bridge for AI agents
│   └── dev-tunnel/              # @spawn-dock/dev-tunnel — WebSocket tunnel client
│
├── .specify/                    # Spec-driven development artifacts
│   ├── memory/
│   │   └── constitution.md      # Project principles (code quality, testing, UX, performance)
│   ├── specs/                   # Feature specifications (one folder per feature)
│   └── templates/               # Document templates used by speckit commands
│
├── .agents/skills/              # AI agent command definitions (speckit)
├── AGENTS.md                    # Agent-level instructions for this workspace
└── WAL.md                       # Write-ahead log (read-only)
```

## Getting Started

### 1. Clone with submodules

```bash
git clone --recurse-submodules git@github.com:SpawnDock/agent.git SpawnDock
# or
git clone --recurse-submodules https://github.com/SpawnDock/agent.git SpawnDock

cd SpawnDock
```

If you already cloned without `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

### 2. Work on a subproject

Each directory under `repo/` is an independent package with its own `package.json`. Navigate into it and follow its own README:

```bash
pnpm install --recursive
```

## Spec-Driven Development Workflow

This workspace uses **speckit** — an AI-assisted workflow for going from idea to implementation through structured documents. All commands are available as slash-commands in Cursor (via `.agents/skills/`).

### Workflow stages

```
/speckit.constitution  →  Establish or update project principles
       ↓
/speckit.clarify       →  Ask clarifying questions about a feature idea
       ↓
/speckit.specify       →  Write a feature specification (user stories + requirements)
       ↓
/speckit.plan          →  Create a technical implementation plan
       ↓
/speckit.tasks         →  Break the plan into a checklist of concrete tasks
       ↓
/speckit.implement     →  Implement tasks one by one
       ↓
/speckit.checklist     →  Generate a QA/review checklist
       ↓
/speckit.taskstoissues →  Export tasks to GitHub Issues
```

You can enter the workflow at any stage. Each command reads the artifacts produced by earlier stages.

### Available commands

| Command                  | What it does                                                                  |
|--------------------------|-------------------------------------------------------------------------------|
| `/speckit.constitution`  | Create or amend the project constitution in `.specify/memory/constitution.md` |
| `/speckit.clarify`       | Generate clarifying questions before writing a spec                           |
| `/speckit.specify`       | Create a feature spec from a plain-language description                       |
| `/speckit.plan`          | Produce a technical design plan for a specified feature                       |
| `/speckit.tasks`         | Break a plan into an ordered task list                                        |
| `/speckit.implement`     | Work through tasks in the task list                                           |
| `/speckit.checklist`     | Generate a QA checklist for a feature or domain                               |
| `/speckit.analyze`       | Analyze existing code for quality, coverage, or design issues                 |
| `/speckit.taskstoissues` | Convert a task list into GitHub Issues                                        |

### Example: starting a new feature

```
/speckit.specify Add a settings screen where users can change their display name
```

This creates `.specify/specs/<number>-<slug>/spec.md` on a new git branch and populates it with prioritized user stories and acceptance criteria.

## Project Constitution

The project's non-negotiable principles live in `.specify/memory/constitution.md`. They govern:

- **Code quality** — zero linter warnings, single responsibility, no dead code
- **Testing standards** — TDD, 80% coverage floor, deterministic tests, CI gate
- **UX consistency** — design tokens, WCAG 2.1 AA, error states, API contract stability
- **Performance** — bundle ≤ 150 KB gzip, API p95 ≤ 500 ms, no N+1 queries

All pull requests must pass the quality gates defined there before merging.

## Feature Specifications

Completed specs live under `.specify/specs/`. Each spec folder corresponds to a git branch and contains:

| File       | Contents                                                            |
|------------|---------------------------------------------------------------------|
| `spec.md`  | User stories, functional requirements, success criteria             |
| `plan.md`  | Technical design and implementation plan (added by `/speckit.plan`) |
| `tasks.md` | Ordered task checklist (added by `/speckit.tasks`)                  |

Current specs:

- [`spawndock-tma-platform`](.specify/specs/spawndock-tma-platform.md) — End-to-end AI-powered TMA creation platform
