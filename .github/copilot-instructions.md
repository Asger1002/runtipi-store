# Copilot Instructions

## Project

A personal app store for [runtipi](https://github.com/meienberger/runtipi) — a self-hosted Docker app manager. Apps are defined as directories under `apps/`, each containing a `config.json`, `docker-compose.yml`, and `metadata/` folder.

## Version Control

This repo uses **jujutsu (`jj`)**, not plain git. Always use `jj` commands for VCS operations.

```bash
jj log                        # View history
jj st                         # Working copy status
jj diff                       # Show changes
jj new                        # Create a new working revision
jj describe -m "message"      # Set the description of the current revision
jj squash                     # Fold working changes into parent commit
jj git push                   # Push to remote
jj git fetch                  # Fetch from remote
```

Never use `git commit`, `git add`, or `git stash` — use `jj` equivalents instead.

## Issue Tracking

This repo uses **beads (`bd`)** for issue tracking. Do **not** use TodoWrite, TaskCreate, markdown TODO lists, or any other task tool.

```bash
bd prime                      # Full workflow context — run at session start
bd ready                      # Find available (unblocked) work
bd show <id>                  # View issue details
bd create "title" -t <type> -p <0-4> -d "description"
bd update <id> --claim        # Claim an issue (atomic)
bd update <id> --status in_progress
bd close <id> --reason "Done"
bd dep add <child-id> <parent-id> --type discovered-from
```

Issue types: `bug`, `feature`, `task`, `epic`, `chore`  
Priorities: `0` (critical) → `4` (backlog)

Use `bd remember` for persistent knowledge instead of MEMORY.md files.

## App Structure

Each app lives under `apps/{id}/` and requires exactly four files:

```
apps/{id}/
├── config.json           # App metadata
├── docker-compose.yml    # Container definition (schema_version 2)
└── metadata/
    ├── description.md    # Markdown description shown in UI
    └── logo.jpg          # Square app icon (1:1 ratio)
```

Key `config.json` fields: `id`, `name`, `port`, `version`, `tipi_version`, `short_desc`, `categories`, `supported_architectures`, `min_tipi_version`, `available`, `exposable`, `dynamic_config`.

`docker-compose.yml` must use schema v2 with `x-runtipi` markers:
```yaml
services:
  {id}:
    image: image:tag
    x-runtipi:
      is_main: true
      internal_port: {container_port}
x-runtipi:
  schema_version: 2
```

## Build & Test

```bash
bun install    # Install validation tooling
bun test       # Validate all apps in apps/ directory
```

## Session Workflow

**At session start:** Run `bd prime` for full context and active work queue.

**At session end** — all steps are mandatory before work is considered complete:

1. File issues for any remaining work
2. Run quality gates if code changed
3. Close finished issues (`bd close <id>`)
4. Push **everything**:
   ```bash
   jj git fetch
   jj rebase -d main          # If needed
   bd dolt push
   jj git push
   ```
5. Verify: working copy is clean and remote is up to date

Work is **not complete** until push succeeds. Never say "ready to push when you are" — always push yourself.
