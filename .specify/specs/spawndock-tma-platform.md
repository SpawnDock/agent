# Feature Specification: SpawnDock — AI-Powered TMA Creation Platform

**Feature Branch**: `feat/spawndock-tma-platform`  
**Created**: 2026-03-21  
**Status**: Draft  
**Input**: User description: "Create an app with SpawnDock - a toolkit that makes TMA creation accessible to everyone, not just experienced programmers."

---

## Overview

SpawnDock is an end-to-end platform that lets any user — regardless of programming experience — turn a plain-language idea into a working Telegram Mini App (TMA). The user never writes code; instead, they describe what they want to an AI agent, which handles all implementation. SpawnDock provides the toolchain, the knowledge base, and the infrastructure that make this possible.

**Component map**:

| Component | Package / Repo | Role |
|-----------|---------------|------|
| Telegram bot | `api` (private) | User entry point — project creation and discovery |
| Bootstrap CLI | `@spawn-dock/cli` | Local project setup and AI agent isolation |
| MCP client | `@spawn-dock/mcp` | Connects AI agent to TMA knowledge base |
| Dev tunnel | `@spawn-dock/dev-tunnel` | Exposes local dev server for real-time preview |
| TMA template | `tma-project` | Next.js + TON Connect + TelegramUI starter |
| API / control plane | `api` (private) | Projects, pairing, tunnels, MCP backend |

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 — First-Time Project Creation via Telegram Bot (Priority: P1)

A non-technical user opens the SpawnDock Telegram bot, creates a new project by describing its name, and receives a single ready-to-run bootstrap command along with a preview URL they can open immediately.

**Why this priority**: This is the front door of the entire product. Every other story depends on a project existing. If this step is broken or confusing, no user reaches any subsequent value.

**Independent Test**: Can be fully tested by sending `/new My Shopping List App` to the bot and verifying a bootstrap command, pairing token, and preview URL are returned — without running any local software.

**Acceptance Scenarios**:

1. **Given** a user has never interacted with the bot, **When** they send `/start`, **Then** they receive a welcome message listing available commands (`/new`, `/launch`, `/help`) in a clear, non-technical tone.
2. **Given** a user sends `/new My Shopping List App`, **When** the bot processes the command, **Then** it creates a project with slug `my-shopping-list-app`, returns the bootstrap shell command, a one-time pairing token (valid 30 minutes), and the preview URL — all in a single message.
3. **Given** a project slug would collide with an existing one, **When** the bot creates the project, **Then** it appends a numeric suffix (e.g., `my-app-2`) and includes the resolved slug in the response.
4. **Given** a user sends `/new` with no title, **When** the bot processes the command, **Then** the project is created with the default title `SpawnDock App` and the user is informed.
5. **Given** an unknown command is sent, **When** the bot processes it, **Then** it returns a helpful error message suggesting `/new` or `/help`.

---

### User Story 2 — Local Environment Bootstrap (Priority: P1)

A user copies the bootstrap command from the bot message, runs it in a terminal on their computer, and within minutes has a fully configured local development environment ready to use — without manually editing any configuration file.

**Why this priority**: The bootstrap step is the only "technical" hurdle. If it fails or requires manual intervention, the product's core promise (accessible to non-programmers) is broken.

**Independent Test**: Can be fully tested by running the bootstrap command on a clean machine and verifying that all config files exist, `npm run dev` starts without errors, and the MCP connection is established.

**Acceptance Scenarios**:

1. **Given** a valid bootstrap command is run, **When** the CLI executes, **Then** it clones `SpawnDock/tma-project`, writes `spawndock.config.json`, `spawndock.dev-tunnel.json`, and `opencode.json` with correct values derived from the pairing token.
2. **Given** the bootstrap command is run, **When** it completes, **Then** the CLI claims the pairing token against the API and receives a `mcpApiKey` and `deviceSecret`, writing them into the local config files — the pairing token is consumed and can no longer be reused.
3. **Given** the pairing token has expired (> 30 minutes old), **When** the bootstrap command is run, **Then** the CLI exits with a clear error message instructing the user to generate a new project in the bot.
4. **Given** Node.js < 20 is detected, **When** bootstrap runs, **Then** the CLI exits early with a human-readable message stating the version requirement and a link to nodejs.org.
5. **Given** the bootstrap succeeds, **When** the user runs `npm run dev`, **Then** Next.js and the dev tunnel start together and the terminal shows the preview URL.

---

### User Story 3 — AI-Driven Feature Development via Chat (Priority: P2)

Inside the bootstrapped project, the user types what they want the app to do in plain language inside the `@spawn-dock/cli` interface. The AI agent (opencode by default) reads the TMA knowledge base via the MCP client and writes the code. The user sees changes live via the preview URL without any intermediate steps.

**Why this priority**: This is the core value proposition. Without it, SpawnDock is just a project scaffold. With it, a non-programmer can build a real app.

**Independent Test**: Can be tested by running the CLI in the bootstrapped project, asking "Add a button that shows a counter", and verifying the change appears in the preview URL within 60 seconds.

**Acceptance Scenarios**:

1. **Given** the dev environment is running, **When** the user types a feature description in the CLI, **Then** the AI agent queries the MCP knowledge base for relevant TMA patterns before writing any code.
2. **Given** the AI writes code, **When** the Next.js dev server detects file changes, **Then** the preview URL reflects the changes within 5 seconds via hot reload.
3. **Given** the AI produces code that introduces a TypeScript error, **When** the dev server reports the error, **Then** the CLI surfaces the compiler error to the user in plain language and the AI attempts to self-correct.
4. **Given** the user asks for something the knowledge base has no context on, **When** the MCP search returns no results, **Then** the AI states its uncertainty rather than hallucinating TMA-specific APIs.
5. **Given** the CLI is run outside a bootstrapped project directory, **When** the user tries to start a session, **Then** the CLI detects missing `spawndock.config.json` and instructs the user to run bootstrap first.

---

### User Story 4 — Real-Time Preview via Dev Tunnel (Priority: P2)

While the user's local dev server is running, anyone with the preview URL (including the user on their phone inside Telegram) can view and interact with the TMA in real time.

**Why this priority**: Real-time feedback is the feedback loop that lets users iterate and validate their ideas without any deployment step. It transforms the product from a code generator into an interactive creation tool.

**Independent Test**: Can be tested by opening the preview URL on a mobile device inside Telegram while `npm run dev` is running, and confirming the TMA renders and responds to interaction.

**Acceptance Scenarios**:

1. **Given** `npm run dev` is running and the tunnel is connected, **When** a preview URL is opened in Telegram, **Then** the TMA loads within 3 seconds on a 4G connection.
2. **Given** the local dev server is stopped, **When** the preview URL is accessed, **Then** the tunnel returns a clear "server offline" message rather than a generic timeout or connection error.
3. **Given** the tunnel is connected, **When** the local server makes a code change, **Then** the tunnel proxies new requests to the updated server without requiring a tunnel restart.
4. **Given** the tunnel WebSocket disconnects unexpectedly, **When** reconnection is attempted, **Then** the tunnel client retries with exponential backoff (max 5 attempts, 30 s max delay) before reporting failure.
5. **Given** multiple devices attempt to connect a tunnel to the same project slug simultaneously, **When** a second connection arrives, **Then** the first connection is gracefully replaced and the project's tunnel session is updated.

---

### User Story 5 — Publishing to GitHub Pages (Priority: P3)

Once the user is satisfied with their TMA, they can publish it to a permanent public URL using a single command or bot instruction. The recommended target is GitHub Pages, with no server infrastructure required.

**Why this priority**: Publishing is the finish line. Without it, the TMA is a local prototype. This story converts the creation experience into a shipped product, but it does not block earlier stories.

**Independent Test**: Can be tested by running the publish command from within the bootstrapped project directory and verifying that a GitHub Pages URL is returned and the TMA is accessible there.

**Acceptance Scenarios**:

1. **Given** the user has a GitHub account and runs the publish command, **When** the command executes, **Then** it builds the Next.js app (`next build` + `next export`), pushes to the `gh-pages` branch of a newly created (or existing) GitHub repo, and returns the public URL.
2. **Given** the user runs the publish command, **When** the build fails due to a code error, **Then** the command surfaces the build error clearly and does not push a broken build.
3. **Given** the app is published, **When** the preview URL is shared inside Telegram, **Then** the TMA opens correctly with Telegram Mini App chrome (header, back button).
4. **Given** the user publishes an update, **When** the `gh-pages` branch already exists, **Then** the publish command replaces the existing deployment without requiring manual branch deletion.

---

### User Story 6 — Project Discovery and Status via Bot (Priority: P3)

A returning user can look up their existing project's preview URL from the Telegram bot without needing to remember it or find it in their local files.

**Why this priority**: Users will close terminals and forget URLs. The bot must be the persistent source of truth for project state.

**Independent Test**: Can be tested by sending `/launch my-app` to the bot after the local tunnel has been stopped, and verifying the response includes the preview URL and the current tunnel status.

**Acceptance Scenarios**:

1. **Given** a user sends `/launch <slug>`, **When** the bot processes it, **Then** it returns the preview URL and indicates whether the tunnel is currently connected or offline.
2. **Given** a user sends `/launch <slug>` for a project they do not own, **When** the bot processes it, **Then** it returns a "not found" error — no information about other users' projects is leaked.
3. **Given** a user sends `/launch` with no slug, **When** the bot processes it, **Then** it returns a helpful error with the correct usage format.

---

### Edge Cases

- What happens when the user's machine goes to sleep while the tunnel is connected? → The tunnel client's heartbeat mechanism should detect the lost connection within 30 s and attempt reconnection.
- What happens when two different users create projects with the same title? → Slug deduplication (numeric suffix) ensures uniqueness; both projects remain independent.
- What happens when the MCP backend is unreachable during a CLI session? → The AI agent falls back to its built-in knowledge and informs the user that TMA-specific guidance is temporarily unavailable.
- What happens when `next export` is not compatible with a server-rendered route in the template? → The publish command detects SSR routes incompatible with static export and warns the user before attempting the build.
- What happens when the pairing token is claimed but the local project directory is lost? → The user must create a new project via `/new`; there is no credential recovery flow in v1.

---

## Requirements *(mandatory)*

### Functional Requirements

**Telegram Bot**

- **FR-001**: The bot MUST support commands `/start`, `/help`, `/new <title>`, and `/launch <slug>`.
- **FR-002**: `/new` MUST create a `Project` record, a `PairingToken` (TTL 30 minutes), and return a bootstrap command, the pairing token value, and the preview URL in a single message.
- **FR-003**: The bot MUST deduplicate project slugs by appending an incrementing numeric suffix when a collision occurs.
- **FR-004**: `/launch` MUST return the preview URL and current tunnel connection status for the caller's own project only.
- **FR-005**: The bot MUST respond to all user messages within 5 seconds; if backend processing exceeds this, it MUST send an acknowledgement message first.

**Bootstrap CLI (`@spawn-dock/cli`)**

- **FR-006**: The CLI MUST clone `SpawnDock/tma-project` and write `spawndock.config.json`, `spawndock.dev-tunnel.json`, and `opencode.json` from a single pairing token argument.
- **FR-007**: The CLI MUST call the API to claim the pairing token and persist the returned `mcpApiKey` and `deviceSecret` into config files; claimed tokens MUST be permanently invalidated.
- **FR-008**: The CLI MUST detect Node.js version and exit with a clear error if < 20.
- **FR-009**: The CLI MUST default to `opencode` as the AI agent runtime; the runtime MUST be configurable via environment variable or config.
- **FR-010**: The CLI MUST isolate the AI agent's filesystem access to the project directory only.

**MCP Client (`@spawn-dock/mcp`)**

- **FR-011**: The MCP client MUST implement a stdio-to-SSE bridge connecting the local AI agent to the SpawnDock MCP backend.
- **FR-012**: The `search` tool MUST query the TMA knowledge base (55 markdown documents) and return ranked, relevant excerpts.
- **FR-013**: The MCP client MUST authenticate all requests to the backend using the `mcpApiKey` from `opencode.json`.
- **FR-014**: The MCP client MUST return a graceful error response (not crash) when the backend is unreachable.

**Dev Tunnel (`@spawn-dock/dev-tunnel`)**

- **FR-015**: The tunnel client MUST establish a WebSocket connection to the SpawnDock control plane using credentials from `spawndock.dev-tunnel.json`.
- **FR-016**: The tunnel MUST proxy HTTP requests from the control plane to the local dev server and return responses.
- **FR-017**: The tunnel client MUST send heartbeats and attempt reconnection with exponential backoff on disconnection (max 5 retries, 30 s ceiling).
- **FR-018**: The control plane MUST replace an existing tunnel session when a new connection arrives for the same project slug.

**API / Control Plane**

- **FR-019**: The API MUST expose a `POST /projects` endpoint for project creation (bot-facing, authenticated by bot secret).
- **FR-020**: The API MUST expose a `POST /projects/:id/claim` endpoint that accepts a pairing token, issues `deviceSecret` and `mcpApiKey`, and marks the token as claimed.
- **FR-021**: The API MUST expose `GET /preview/:slug/*` that proxies requests through the active tunnel session for the given slug.
- **FR-022**: The API MUST expose an SSE endpoint at `/sse` for MCP protocol communication.
- **FR-023**: The API MUST rate-limit MCP requests to 10 req/s per IP.
- **FR-024**: Project and credential state MUST be persisted to disk (JSON store) and survive server restarts.

**TMA Template (`tma-project`)**

- **FR-025**: The template MUST include Next.js, TypeScript, TON Connect, `@telegram-apps/sdk`, and TelegramUI out of the box.
- **FR-026**: The template MUST include a `mockTelegramEnv` shim so the app renders correctly in a browser (non-Telegram) during development.
- **FR-027**: `pnpm run dev` MUST start Next.js and the dev tunnel concurrently with a single command.

### Key Entities

- **Project**: A user's TMA project. Attributes: `id`, `slug` (unique URL-safe identifier), `title`, `templateId`, `ownerTelegramId`, `createdAt`, `status` (`draft` | `active`).
- **PairingToken**: A one-time token linking a bot-created project to a local bootstrap. Attributes: `token`, `projectId`, `createdAt`, `expiresAt` (30 min TTL), `claimedAt`, `claimedDeviceId`. Consumed on first claim.
- **DeviceCredential**: Long-lived credential for a bootstrapped local environment. Attributes: `id`, `projectId`, `label`, `secret` (for tunnel auth), `mcpApiKey` (for MCP auth), `createdAt`, `revokedAt`.
- **TunnelSession**: Active tunnel connection state. Attributes: `id`, `projectId`, `deviceId`, `localOrigin`, `connectedAt`, `lastHeartbeatAt`, `status` (`connected` | `disconnected`).

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A non-programmer can go from `/new` in the bot to a visible, interactive TMA in their Telegram client in under **10 minutes** on a first attempt, following written instructions only.
- **SC-002**: Bootstrap command succeeds on a clean macOS or Linux machine with Node.js 20+ and internet access in under **3 minutes** (excluding git clone network time).
- **SC-003**: Preview URL loads the TMA in Telegram within **3 seconds** on a 4G mobile connection while the local dev server is running.
- **SC-004**: AI agent code changes are reflected in the preview URL within **5 seconds** via hot reload.
- **SC-005**: The tunnel client reconnects automatically after a network interruption within **30 seconds**.
- **SC-006**: At least **80%** of unit test coverage is maintained across all packages; critical paths (token claim, tunnel auth, MCP auth) require **100%** branch coverage.
- **SC-007**: Initial TMA bundle delivered to the user's device is **≤ 150 KB** gzipped JavaScript.
- **SC-008**: API read endpoints (project lookup, preview proxy header processing) respond at p95 **≤ 500 ms** under nominal load.
- **SC-009**: Zero personally identifiable information beyond Telegram user IDs is stored in the API state store; Telegram IDs are used only for project ownership checks.
- **SC-010**: **90%** of users who successfully bootstrap can publish their app to GitHub Pages without additional support, given the publish command documentation.
