# SpawnDock Workspace

**Development workspace for the SpawnDock platform.**
Combines all components as git submodules with a spec-driven development workflow powered by **speckit**.

---

## Repository layout

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'primaryColor': '#ff8c00', 'primaryTextColor': '#1a1a1a', 'primaryBorderColor': '#e67300', 'lineColor': '#ff a500', 'secondaryColor': '#fff3e0', 'tertiaryColor': '#ffe0b2', 'edgeLabelBackground': '#fff8f0', 'clusterBkg': '#fff8f0', 'clusterBorder': '#ff8c00'}}}%%
graph LR
  subgraph repo["repo/ — submodules"]
    api["api\nTelegram bot + MCP server\n+ tunnel control plane"]
    tma["tma-project\nNext.js + TON Connect\n+ Telegram UI"]
    mcp["mcp-client\nstdio-to-SSE bridge\nfor AI agents"]
    tunnel["dev-tunnel\nWebSocket tunnel client"]
    cli["cli\nAI runtime launcher"]
    create["create\nBootstrap CLI"]
  end

  subgraph specify[".specify/ — speckit"]
    memory["memory/\nconstitution.md"]
    specs["specs/\nfeature specifications"]
    templates["templates/\ndocument templates"]
  end

  subgraph agents[".agents/"]
    skills["skills/\nAI agent commands"]
  end

  style api fill:#ff8c00,stroke:#e67300,color:#fff
  style tma fill:#ff8c00,stroke:#e67300,color:#fff
  style mcp fill:#ff8c00,stroke:#e67300,color:#fff
  style tunnel fill:#ff8c00,stroke:#e67300,color:#fff
  style cli fill:#ff8c00,stroke:#e67300,color:#fff
  style create fill:#ff8c00,stroke:#e67300,color:#fff
  style memory fill:#ffb74d,stroke:#e67300,color:#1a1a1a
  style specs fill:#ffb74d,stroke:#e67300,color:#1a1a1a
  style templates fill:#ffb74d,stroke:#e67300,color:#1a1a1a
  style skills fill:#ffe0b2,stroke:#e67300,color:#1a1a1a
```

---

## Getting started

### 1. Clone with submodules

```bash
git clone --recurse-submodules git@github.com:SpawnDock/agent.git SpawnDock
cd SpawnDock
```

If already cloned without `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

### 2. Install dependencies

```bash
pnpm install --recursive
```

Each directory under `repo/` is an independent package — navigate into it and follow its own README.

---

## Spec-driven development

This workspace uses **speckit** — an AI-assisted workflow for going from idea to implementation through structured documents. All commands are slash-commands in Cursor (via `.agents/skills/`).

### Workflow

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'primaryColor': '#ff8c00', 'primaryTextColor': '#fff', 'primaryBorderColor': '#e67300', 'lineColor': '#ff8c00', 'secondaryColor': '#fff3e0', 'tertiaryColor': '#ffe0b2'}}}%%
flowchart TD
  A["/speckit.constitution\nEstablish project principles"] --> B
  B["/speckit.clarify\nAsk clarifying questions"] --> C
  C["/speckit.specify\nWrite feature spec"] --> D
  D["/speckit.plan\nCreate implementation plan"] --> E
  E["/speckit.tasks\nBreak into task checklist"] --> F
  F["/speckit.implement\nImplement tasks"] --> G
  G["/speckit.checklist\nQA / review checklist"] --> H
  H["/speckit.taskstoissues\nExport to GitHub Issues"]

  style A fill:#ff8c00,stroke:#e67300,color:#fff
  style B fill:#ff9800,stroke:#e67300,color:#fff
  style C fill:#ffa726,stroke:#e67300,color:#fff
  style D fill:#ffb74d,stroke:#e67300,color:#1a1a1a
  style E fill:#ffcc80,stroke:#e67300,color:#1a1a1a
  style F fill:#ffb74d,stroke:#e67300,color:#1a1a1a
  style G fill:#ffa726,stroke:#e67300,color:#fff
  style H fill:#ff9800,stroke:#e67300,color:#fff

  linkStyle default stroke:#ff8c00,stroke-width:2px
```

You can enter the workflow at any stage. Each command reads artifacts produced by earlier stages.

### Commands

| Command | Description |
| :--- | :--- |
| `/speckit.constitution` | Create or amend project principles |
| `/speckit.clarify` | Generate clarifying questions before writing a spec |
| `/speckit.specify` | Create a feature spec from a plain-language description |
| `/speckit.plan` | Produce a technical design plan |
| `/speckit.tasks` | Break a plan into an ordered task list |
| `/speckit.implement` | Work through tasks in the task list |
| `/speckit.checklist` | Generate a QA checklist |
| `/speckit.analyze` | Analyze code for quality, coverage, or design issues |
| `/speckit.taskstoissues` | Convert a task list into GitHub Issues |

### Example

```
/speckit.specify Add a settings screen where users can change their display name
```

Creates `.specify/specs/<number>-<slug>/spec.md` on a new branch with prioritized user stories and acceptance criteria.

---

## Project constitution

Non-negotiable principles in `.specify/memory/constitution.md`:

| Area | Standards |
| :--- | :--- |
| **Code quality** | Zero linter warnings, single responsibility, no dead code |
| **Testing** | TDD, 80 % coverage floor, deterministic tests, CI gate |
| **UX** | Design tokens, WCAG 2.1 AA, error states, API contract stability |
| **Performance** | Bundle ≤ 150 KB gzip, API p95 ≤ 500 ms, no N+1 queries |

All pull requests must pass these quality gates before merging.

---

## Feature specifications

Specs live under `.specify/specs/`. Each folder maps to a git branch:

| File | Contents |
| :--- | :--- |
| `spec.md` | User stories, functional requirements, success criteria |
| `plan.md` | Technical design and implementation plan |
| `tasks.md` | Ordered task checklist |

Current specs:

- [`spawndock-tma-platform`](.specify/specs/spawndock-tma-platform.md) — End-to-end AI-powered TMA creation platform
