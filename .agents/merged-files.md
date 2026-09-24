# Merged Files

This file documents files that were merged or adapted when integrating the runtipi example-appstore template.

## .gitignore
- **Our content**: Beads/Dolt entries (`.dolt/`, `*.db`, `.beads-credential-key`)
- **Template content**: `node_modules/`
- **Result**: Both sections combined

## __tests__/apps.test.ts
- **Source**: https://github.com/runtipi/example-appstore/blob/main/__tests__/apps.test.ts
- **Change**: Template checked for `docker-compose.json`; changed to `docker-compose.yml` because new apps use schema_version 2 yml format
- **Removed**: The entire `docker-compose.json` validation describe block (not applicable to schema v2 apps)

## README.md
- **Template README**: Generic "Example App Store Template" boilerplate — discarded
- **Our README**: Kept as-is (describes this specific store)

## renovate.json
- **Template**: Matched `docker-compose.json` files
- **Ours**: Updated regex to match `docker-compose.yml` files (consistent with test fix)
