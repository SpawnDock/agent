# npm publishing (SpawnDock)

All packages are published from **GitHub Actions**, not from a developer laptop.

## Repository secret

Configure **`NPM_KEY`** — npm automation token with publish rights to `@spawn-dock/*` (used by all publish workflows).

## Workflows

| Repository | Workflow | Trigger |
|------------|----------|---------|
| `mcp-client` | `.github/workflows/publish.yml` | Push to `main`, tag `v*`, or **Run workflow** |
| `dev-tunnel` | same pattern | same |
| `create-spawn-dock` | same pattern | same |
| `cli` | `.github/workflows/release.yml` | After **Check** succeeds on `main` (`action-release` publishes `spawndock-cli` and `docker-git` app) |

## Prerelease versions (`1.0.0-beta.x`)

- **Tag-driven release:** create an annotated tag whose name matches the version, e.g. `v1.0.0-beta.1`, and push it. The publish job sets `package.json` version from the tag and runs `npm publish --tag beta` for prerelease semver.
- **Canary from `main`:** merge to `main` publishes a `canary` dist-tag (version in `package.json` + timestamp + short SHA).

## Install

- Stable / tagged prerelease on `beta`: `npm install @spawn-dock/mcp@beta` (or exact version).
- Canary: `npm install @spawn-dock/mcp@canary`.
