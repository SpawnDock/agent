---
name: syncing-project
description: Synchronizes the SpawnDock monorepo to its current state: pulls latest changes, detects and initializes new submodules, updates existing ones, verifies project integrity, and auto-fixes clear errors. For ambiguous errors, brainstorms solutions from specs and offers options. Use when the user asks to check or update project actuality, sync the project, update submodules, or says the project may be outdated.
---

# Syncing Project

## Overview

This skill brings the entire SpawnDock monorepo (root + all `repo/*` submodules) up to date and verifies it is internally consistent.

## Workflow

Copy and track this checklist:

```
- [ ] Step 1: Pull root repo
- [ ] Step 2: Detect new / removed submodules
- [ ] Step 3: Update all submodules
- [ ] Step 4: Verify integrity
- [ ] Step 5: Fix clear errors
- [ ] Step 6: Brainstorm ambiguous errors (if any)
- [ ] Step 7: Report final state
```

---

## Step 1: Pull root repo

```bash
git -C <workspace_root> pull --rebase --autostash
```

If conflicts appear on `.gitmodules` → resolve by accepting the incoming version, then continue.

---

## Step 2: Detect new / removed submodules

```bash
git -C <workspace_root> submodule sync
```

This updates `.git/config` with any new URLs from `.gitmodules`. List what changed:

```bash
git -C <workspace_root> config --file .gitmodules --get-regexp path
```

Compare against `git submodule status` to surface any newly registered but uninitialised entries.

---

## Step 3: Update all submodules

**Initialize and pull to pinned commits** (safe, preserves what the root commit says):

```bash
git -C <workspace_root> submodule update --init --recursive
```

**If the user asked to pull the latest branch tips** (not just pinned commits):

```bash
git -C <workspace_root> submodule update --init --recursive --remote
```

Check for merge conflicts inside each submodule after this step:

```bash
git -C <workspace_root> submodule foreach 'git status --short'
```

---

## Step 4: Verify integrity

Run the following checks and collect all failures before proceeding:

### 4a. Submodule HEAD validity
```bash
git -C <workspace_root> submodule status
```
Lines starting with `-` = not initialised. Lines starting with `+` = detached at non-matching commit.

### 4b. Package dependencies (for each submodule that has `package.json`)
```bash
git -C <workspace_root> submodule foreach '[ -f package.json ] && npm install --silent || true'
```

### 4c. TypeScript / lint (for each submodule with a `tsconfig.json`)
```bash
git -C <workspace_root> submodule foreach '[ -f tsconfig.json ] && npx tsc --noEmit 2>&1 | head -40 || true'
```

Collect the full list of errors from all checks before fixing anything.

---

## Step 5: Fix clear errors

Apply these fixes automatically without asking the user:

| Error | Fix |
|-------|-----|
| Uninitialised submodule (`-` prefix) | `git submodule update --init <path>` |
| Missing `node_modules` | `npm install` inside the submodule |
| Lock-file conflict (`package-lock.json`) | `npm install` regenerates it |
| Submodule detached at wrong commit (`+` prefix) AND `--remote` was NOT used | `git submodule update <path>` to re-pin |
| TypeScript error from a missing generated file | Run the codegen script declared in `package.json` scripts |

After each fix, re-run the failing check to confirm resolution before moving on.

---

## Step 6: Brainstorm ambiguous errors

If an error has no clear automatic fix (e.g., breaking API change between submodules, spec mismatch, unknown dependency conflict), **do not guess**.

1. Read the relevant spec in `.specify/specs/` and the constitution in `.specify/memory/constitution.md`.
2. Invoke the **brainstorming skill** (`.cursor/skills/brainstorming/SKILL.md`) to explore options.
3. Present the user with **2–4 concrete options**, each with:
   - What it changes
   - Trade-offs
   - Estimated effort

Do not implement any option until the user selects one.

---

## Step 7: Report final state

Summarise the sync result:

```
Sync complete
==============
Root repo:        main @ <short-sha> (up to date)
New submodules:   <list or "none">
Updated:          <list of submodule: old-sha → new-sha>
Errors fixed:     <list or "none">
Pending decisions:<list or "none">
```

If any pending decisions remain, remind the user that those submodules are in a degraded state until resolved.
