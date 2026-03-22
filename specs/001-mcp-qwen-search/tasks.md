---
description: "Task list for MCP containerized Qwen search (001-mcp-qwen-search)"
---

# Tasks: MCP Knowledge Search via Containerized CLI

**Input**: Design documents from `/specs/001-mcp-qwen-search/`  
**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md)

**Tests**: Included for failure paths and runner contract (aligns with project constitution: tests for functional changes).

**Organization**: Phases follow user stories P1 → P2 → P3 from [spec.md](./spec.md).

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no unmet dependencies)
- **[Story]**: `[US1]` … `[US3]` for user-story phases only

## Phase 1: Setup (shared infrastructure)

**Purpose**: Container image and operator-facing docs for the Qwen CLI sandbox.

- [x] T001 Add OCI image definition for Qwen Code CLI in `repo/api/docker/qwen-search/Dockerfile`
- [x] T002 [P] Document build, run, and volume contract in `repo/api/docker/qwen-search/README.md`
- [x] T003 [P] Add new environment variables for container mode to `repo/api/.env.example`

---

## Phase 2: Foundational (blocking prerequisites)

**Purpose**: Configuration and container runner primitives — **no user story work before this completes**.

- [x] T004 Extend typed config for `QWEN_MODE=container` and image/mount/limits in `repo/api/src/config.ts`
- [x] T005 Implement `runQwenSearchInContainer` (or equivalent) with timeout and max output size in `repo/api/src/qwen/container-runner.ts`
- [x] T006 [P] Add correlation-id generation and attach to errors/logs in `repo/api/src/qwen/container-runner.ts`
- [x] T007 [P] Ensure runner builds `docker`/`podman` argv with **read-only** bind mount for host knowledge path in `repo/api/src/qwen/container-runner.ts`

**Checkpoint**: Host can invoke image with `:ro` mount in dev (manual smoke per README).

---

## Phase 3: User Story 1 — MCP search returns grounded answer (Priority: P1) — MVP

**Goal**: MCP `search` tool uses the container path and returns answer + machine-readable source paths; corpus files on host unchanged.

**Independent Test**: Call MCP `search` with a fixed query against a temp copy of `repo/api/knowledge`; assert non-empty answer, expected sources, and stable file hashes pre/post.

### Tests for User Story 1

- [x] T008 [P] [US1] Add unit tests with mocked container runner returning fixed JSON stdout in `repo/api/src/__tests__/qwen-container.test.ts`
- [x] T009 [P] [US1] Add test that runner command includes read-only mount flag for corpus path in `repo/api/src/__tests__/qwen-container.test.ts`

### Implementation for User Story 1

- [x] T010 [US1] Branch `QWEN_MODE=container` in `repo/api/src/mcp.ts` to use container runner + existing `searchLocalKnowledgeContext` for match context
- [x] T011 [US1] Pipe container stdout through `extractQwenCodeResult` / shared parser in `repo/api/src/mcp.ts` and `repo/api/src/qwen/code.ts` (refactor if needed to avoid duplication)
- [x] T012 [US1] Normalize and validate `sources[].file` paths against `repo/api/knowledge` roots in `repo/api/src/mcp.ts` or `repo/api/src/qwen/parser.ts`
- [x] T013 [US1] Preserve existing `http` / legacy `cli` modes when `QWEN_MODE` is not `container` in `repo/api/src/mcp.ts`

**Checkpoint**: MVP — container-backed MCP search works in dev with Docker.

---

## Phase 4: User Story 2 — Graceful degradation (Priority: P2)

**Goal**: Timeouts, non-zero exits, and oversized output surface as MCP tool errors; API/MCP process stays alive; no secret leakage in logs.

**Independent Test**: Mock runner to throw/timeout; assert MCP error shape and that a second `search` call still succeeds.

### Tests for User Story 2

- [x] T014 [P] [US2] Add timeout and non-zero exit tests for container runner in `repo/api/src/__tests__/qwen-container.test.ts`
- [x] T015 [P] [US2] Extend `repo/api/src/__tests__/mcp.test.ts` to cover `QWEN_MODE=container` error paths without crashing the server

### Implementation for User Story 2

- [x] T016 [US2] Map runner failures to stable MCP tool error content (message + correlation id) in `repo/api/src/mcp.ts`
- [x] T017 [US2] Implement stdout/stderr size caps with clear errors in `repo/api/src/qwen/container-runner.ts`
- [x] T018 [P] [US2] Redact env values matching token/key patterns from runner logs in `repo/api/src/qwen/container-runner.ts`

**Checkpoint**: Failure injection suite green; unrelated MCP tools still callable after errors.

---

## Phase 5: User Story 3 — Operators refresh corpus without new image (Priority: P3)

**Goal**: Documentation on host can change; search sees updates after standard redeploy/restart; unreadable corpus entries do not crash search.

**Independent Test**: Add unique markdown under `knowledge/` in fixture dir, rerun search after simulated remount (or process restart in test).

### Tests for User Story 3

- [x] T019 [P] [US3] Add corpus walker test skipping unreadable entries in `repo/api/src/__tests__/local-search.test.ts` (or `repo/api/src/__tests__/knowledge-walker.test.ts`)

### Implementation for User Story 3

- [x] T020 [US3] Document rolling update / volume refresh procedure in `repo/api/docker/qwen-search/README.md` and link from `repo/api/README.md`
- [x] T021 [US3] Harden `loadKnowledgeDocuments` / ranking against symlinks and oversize files in `repo/api/src/local-search.ts`

**Checkpoint**: Ops path documented; robust corpus scanning.

---

## Phase 6: Polish & cross-cutting

**Purpose**: Benchmark harness, docs, regression gate.

- [x] T022 [P] Add optional benchmark query list (20+) for SC-001/SC-004 in `specs/001-mcp-qwen-search/benchmark-queries.md` referencing corpus snapshot strategy in `specs/001-mcp-qwen-search/plan.md`
- [x] T023 [P] Update top-level `repo/api/README.md` with when to use `container` vs `http` vs `cli` modes
- [x] T024 Run `npm test` (script in `repo/api/package.json`) from `repo/api/` and fix regressions in touched files under `repo/api/src/`

---

## Dependencies & execution order

### Phase dependencies

| Phase | Depends on |
|-------|------------|
| 1 Setup | — |
| 2 Foundational | Phase 1 (image + env contract documented) |
| 3 US1 | Phase 2 |
| 4 US2 | Phase 3 (happy path exists) |
| 5 US3 | Phase 3 (can parallelize parts with US2 after T010) |
| 6 Polish | All targeted user stories done |

### User story dependencies

- **US1**: After Foundational — no dependency on US2/US3.
- **US2**: After US1 core path (T010–T012) so errors wrap real flow.
- **US3**: Largely independent of US2; can start after Phase 2 for docs (T020) but T021 benefits from stable US1 behavior.

### Parallel opportunities

- **Phase 1**: T002 and T003 parallel with T001 after Dockerfile exists.
- **Phase 2**: T006 and T007 parallel after T005 skeleton.
- **US1 tests**: T008 and T009 parallel.
- **US2 tests**: T014 and T015 parallel.
- **Polish**: T022 and T023 parallel.

---

## Parallel example: User Story 1

```bash
# After T010 is merged locally, two agents can:
# Agent A: T008 + T009 in repo/api/src/__tests__/qwen-container.test.ts
# Agent B: T012 path validation in repo/api/src/mcp.ts
```

---

## Implementation strategy

### MVP first (User Story 1 only)

1. Complete Phase 1–2.  
2. Complete Phase 3 (US1) through T013.  
3. Stop and validate MCP `search` with Docker + read-only mount.  

### Incremental delivery

1. Add US2 (failure envelope + tests).  
2. Add US3 (ops docs + corpus hardening).  
3. Polish + full `npm test`.  

---

## Task summary

| Metric | Count |
|--------|------:|
| Total tasks | 24 |
| Phase 1 | 3 |
| Phase 2 | 4 |
| US1 | 6 |
| US2 | 5 |
| US3 | 3 |
| Polish | 3 |

**Suggested MVP scope**: Phases 1–3 (T001–T013).
