# Hermes Workspace

A web-based development environment for [Hermes Agent](https://hermes-agent.nousresearch.com), by outsourc-e. Hermes Workspace provides a collaborative UI for building, testing, and managing your Hermes Agent setup — including skill development, session inspection, and configuration management — all from your browser.

## Architecture

Hermes Workspace runs as a standalone container that connects to your existing **Hermes Agent** gateway via its OpenAI-compatible API endpoint. The two apps are deployed independently in Runtipi and communicate over the Docker host network.

| Service | Role | Port |
|---------|------|------|
| **hermes-workspace** | Web UI for workspace management | 3000 |
| **hermes-agent** | Gateway backend (AI agent) | 8642 (API) |

## Prerequisites

This app requires a running **Hermes Agent** installation with its API server enabled and reachable. The API URL is configured via the `HERMES_API_URL` environment variable (set during installation in the form below).

If your Hermes Agent gateway does **not** have API authentication enabled (which is the default for this store's setup), `HERMES_ALLOW_INSECURE_REMOTE` is set automatically. If you later enable an API server key, you will need to provide it via the form and disable insecure remote mode.

## Connecting to Hermes Agent

By default, the workspace connects to `http://host.docker.internal:8642` — this targets the Hermes Agent API published on the host at the default port. If you changed the gateway API port during Hermes Agent installation, update the URL to match.

On Linux, an `extra_hosts` entry is automatically configured so that `host.docker.internal` resolves correctly inside the container.

## Links

- [Hermes Agent Docker Guide](https://hermes-agent.nousresearch.com/docs/user-guide/docker)