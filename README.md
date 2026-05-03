# runtipi-store

A personal app store for [Runtipi](https://github.com/meienberger/runtipi).

## Apps

- **whoami** - Tiny Go server that prints OS info and HTTP request details (for testing)
- **hermes-agent** - Self-hosted AI agent with web dashboard (Nous Research)

## Adding a new app

Each app needs: `apps/{id}/config.json`, `apps/{id}/docker-compose.yml`, `apps/{id}/metadata/logo.jpg`, `apps/{id}/metadata/description.md`.

Run `bun test` to validate.
