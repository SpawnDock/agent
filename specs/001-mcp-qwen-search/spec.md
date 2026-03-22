# Feature Specification: MCP Knowledge Search via Containerized CLI

**Feature Branch**: `001-mcp-qwen-search`  
**Created**: 2026-03-22  
**Status**: Draft  
**Input**: User description: Containerized Qwen Code CLI as the search engine for the SpawnDock knowledge base, invoked through MCP, with documentation mounted read-only into the container; reuse existing API-hosted knowledge rather than a new repository.

## Context & Alignment

This feature extends the SpawnDock platform described in [spawndock-tma-platform](../../.specify/specs/spawndock-tma-platform.md), specifically the expectation that an AI agent queries a **knowledge base via MCP** before writing TMA code. Today, search/fallback may use simpler local ranking; this specification defines a **constrained, container-isolated CLI search path** so that advanced code-aware retrieval can run without granting write access to corpus files.

**Assumptions** (until product overrides them):

- The **canonical knowledge corpus** for MCP is the same documentation tree already shipped or mounted with the **API** service (no separate documentation repository is required for v1).
- The **Qwen Code CLI** is treated as a **black-box** tool: the host process sends a **textual request** and collects a **textual response**; exact CLI flags and prompts are implementation details for `/speckit.plan`.
- **Credentials** needed by the CLI (if any) are supplied by the deployment environment (e.g. environment variables) and are **never** written into the mounted knowledge directory.

## User Scenarios & Testing *(mandatory)*

### User Story 1 — MCP Search Returns an Answer From the Knowledge Corpus (Priority: P1)

An AI client calls the MCP search capability with a natural-language question. The system runs an isolated search process that may only **read** the mounted knowledge files, then returns a **single consolidated answer** (and enough traceability to see which parts of the corpus were relevant).

**Why this priority**: This is the core value: grounded answers for agents building TMAs without exposing mutable knowledge to the search tool.

**Independent Test**: Invoke the MCP search tool with a fixed query against a **frozen** snapshot of knowledge files in a test environment; assert the response is non-empty, references expected concepts, and that **no file in the corpus changes** (size, hash, or mtime).

**Acceptance Scenarios**:

1. **Given** a populated knowledge directory mounted for search, **When** the MCP client issues a search with a clear question, **Then** the response contains an answer suitable for downstream agent consumption and does not error under nominal conditions.
2. **Given** the same query issued twice without corpus changes, **When** both invocations complete successfully, **Then** results are deterministic enough for automated testing (e.g. same cited sources or stable ordering policy — exact fuzziness of natural-language answers is bounded by a documented tolerance in the plan phase).
3. **Given** a successful search, **When** an auditor compares corpus file checksums before and after the call, **Then** **all** knowledge files are **bit-for-bit identical** to their pre-call state.

---

### User Story 2 — Graceful Degradation When the Search Tool Fails (Priority: P2)

If the isolated search process crashes, times out, or returns unusable output, the MCP layer surfaces a **clear failure** to the client without taking down the wider API or MCP server process.

**Why this priority**: Container/CLI failures must not become platform outages; agents need a predictable error envelope.

**Independent Test**: Simulate process exit, timeout, and empty stdout; assert the MCP tool returns a structured error and the host service remains healthy for unrelated endpoints/tools.

**Acceptance Scenarios**:

1. **Given** the search subprocess exceeds a configured maximum runtime, **When** the host waits for completion, **Then** the call ends with a timeout error and resources (subprocess/container) are released.
2. **Given** the search subprocess exits with a non-zero status, **When** the host collects stderr/stdout, **Then** the MCP client receives an error that includes a correlation id or safe diagnostic summary **without** leaking secrets.
3. **Given** a failed search, **When** subsequent unrelated MCP requests are made, **Then** they still succeed within normal latency bounds.

---

### User Story 3 — Operators Refresh Knowledge Without Rebuilding the Search Image (Priority: P3)

Operations can **update documentation files** on the host (or in the API deployment artifact) and have new content visible to search **without** publishing a new container image, by remounting or replacing the read-only volume source.

**Why this priority**: Keeps documentation velocity high and aligns with “single source of truth” in the API repo.

**Independent Test**: Add a new markdown file to the mounted directory, rerun search with a query unique to that file, observe the new content reflected in the answer or citations.

**Acceptance Scenarios**:

1. **Given** an updated file in the host knowledge directory, **When** the deployment refreshes the mount (restart or rolling update as defined in the plan), **Then** a search query targeting the new material returns results that include that material.
2. **Given** a malformed or non-text file appears in the corpus, **When** search runs, **Then** the tool skips or safely ignores unreadable entries and still returns a best-effort answer or a controlled error (no crash).

---

### Edge Cases

- Empty or whitespace-only queries.
- Extremely long queries or responses; policy for truncation and client notification.
- Concurrent overlapping search requests — isolation and fairness (queue vs parallel limits).
- Corpus contains symlinks, very large files, or pathological directory depth.
- Locale / encoding issues in corpus files (UTF-8 assumed; document fallback).
- Resource exhaustion (CPU, memory) inside the container — back-pressure or refusal with explicit error.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The platform MUST expose a **search** operation through the **existing MCP** surface used by SpawnDock agents.
- **FR-002**: Each search operation MUST send a **textual input** (the user/agent query) to a **CLI-based search process** and MUST return **textual output** as the primary search result to the MCP client.
- **FR-003**: The CLI search process MUST run **inside a container** (or container-equivalent isolation defined in planning) so that filesystem and process boundaries are enforceable by the runtime.
- **FR-004**: The container MUST receive the knowledge corpus as a **mounted directory** containing the **same documentation assets** that back MCP knowledge today (hosted with / managed alongside the **API** codebase), avoiding a separate documentation repository for this feature’s first version.
- **FR-005**: The mounted knowledge path inside the container MUST be **read-only** at the filesystem level (e.g. read-only bind mount); the CLI process MUST NOT require write permission to the corpus for normal operation.
- **FR-006**: The system MUST enforce **no writes** from the search process to corpus files: after any search call, corpus content on the host MUST remain unchanged (see User Story 1).
- **FR-007**: The MCP response MUST include a machine-readable list of **corpus file paths** used or cited for the answer; section- or heading-level pointers are required when the integration can derive them from CLI output or file structure, otherwise file-level paths suffice.
- **FR-008**: The host integration MUST enforce **upper bounds** on runtime and output size for a single search call to protect shared services.
- **FR-009**: Failures of the CLI/container MUST be translated into **MCP-visible errors** that do not crash unrelated MCP tools or core API routes.

### Security & Isolation Requirements

- **SR-001**: Principle of least privilege: the container identity MUST NOT have credentials to mutate production databases or secrets unrelated to read-only search, except what is strictly required for the CLI to function.
- **SR-002**: Search logs MUST redact or omit sensitive environment values and tokens.

### Key Entities

- **Search request**: Natural-language (or keyword) query text; optional limits (timeout, max sources) to be finalized in planning.
- **Search response**: Primary answer text; list of **source references** tied to corpus paths; optional raw excerpts.
- **Knowledge corpus**: Versioned set of files under the API-managed documentation tree; treated as immutable during search calls.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: For a benchmark suite of at least **20** fixed queries against a fixed corpus snapshot, **100%** of successful invocations return a non-empty answer body within an agreed wall-clock budget (to be set in planning, e.g. p95 under **30 s** for interactive agent use).
- **SC-002**: Automated integrity checks demonstrate **zero** unauthorized modifications to corpus files across **500** consecutive search invocations in stress testing.
- **SC-003**: In failure-injection tests (timeout, non-zero exit), **100%** of cases return a structured MCP error and **0%** cause the parent MCP server process to exit.
- **SC-004**: For **90%** of benchmark queries, at least one returned source reference maps to a real corpus path present in the snapshot (verifiable automatically).

## Out of Scope (v1)

- Authoring or editing knowledge files through MCP or the search container.
- Hosting a second, divergent documentation repository solely for this feature.
- Guaranteeing semantic correctness of answers beyond “grounded in mounted files” — quality tuning is iterative but not a v1 contractual metric beyond SC-001/SC-004.

## Dependencies

- Existing MCP server and tool registration in the **API** service.
- Existing knowledge directory layout consumed by MCP today.
- Container runtime available in target deployment environments (local dev + production parity goal).
