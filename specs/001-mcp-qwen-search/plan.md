# Implementation Plan: MCP Knowledge Search via Containerized CLI

**Feature**: `001-mcp-qwen-search`  
**Spec**: [spec.md](./spec.md)  
**Primary codebase**: `repo/api` (SpawnDock control plane + MCP server)

## Summary

Run **Qwen Code CLI** inside a **container** with the existing **`knowledge/`** tree bind-mounted **read-only**. The **MCP `search` tool** in `repo/api/src/mcp.ts` invokes the container (instead of or as an alternative to the current host-local `QWEN_MODE=cli` `spawn` path), parses JSON output via existing `repo/api/src/qwen/parser.ts` / `code.ts` patterns, and returns **answer + source paths** without mutating corpus files on the host.

## Technical Stack

| Area | Choice |
|------|--------|
| Host runtime | Node.js + TypeScript (`repo/api`) |
| MCP | Existing `@modelcontextprotocol/sdk` wiring in `repo/api/src/mcp.ts` |
| Corpus | `repo/api/knowledge/**` (same tree as `local-search.ts` uses via `src/knowledge` resolution) |
| Isolation | OCI container (Docker / compatible runtime) |
| CLI | `qwen` (or image entrypoint), configured via env (`QWEN_CODE_COMMAND`, timeouts, auth flags) |

## Project Structure (additions / touch points)

| Path | Role |
|------|------|
| `repo/api/docker/qwen-search/Dockerfile` (or `repo/api/infra/...`) | Image installing Qwen Code CLI and minimal deps |
| `repo/api/docker/qwen-search/README.md` | Build/run contract for dev and CI |
| `repo/api/src/qwen/container-runner.ts` (new) | `docker run` (or `podman`) with `:ro` mount, stream collect, timeout, max output bytes |
| `repo/api/src/config.ts` | New env: e.g. `QWEN_MODE=container`, `QWEN_CONTAINER_IMAGE`, `QWEN_CONTAINER_KNOWLEDGE_MOUNT`, `QWEN_CONTAINER_*` limits |
| `repo/api/src/mcp.ts` | Branch container path; preserve HTTP + optional legacy CLI behavior per config |
| `repo/api/src/qwen/code.ts` | Reuse `queryQwenCode` prompt/parse or thin adapter for container stdout |
| `repo/api/src/__tests__/` | Unit tests with mocked runner; integration optional behind `CI` / docker flag |

## Out of Scope (implementation)

- Changing knowledge authoring workflow beyond read-only mount refresh.
- Non-Docker runtimes unless trivial alias to same CLI contract.

## Risks

- Qwen CLI version drift inside image vs host.
- Docker socket availability in production (document requirement or use rootless / k8s job pattern in later iteration).

## Benchmark queries

Optional fixed query list for SC-001 / SC-004-style checks: [benchmark-queries.md](./benchmark-queries.md) (same `knowledge/` snapshot revision when comparing runs).
