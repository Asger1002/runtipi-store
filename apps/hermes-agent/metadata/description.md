runtipi-store/apps/hermes-agent/metadata/description.md
```
# Hermes Agent

A self-hosted AI agent by [Nous Research](https://nousresearch.com). Hermes runs as a persistent gateway that connects to your preferred chat platform (Telegram, Discord, Slack, WhatsApp) and/or exposes an OpenAI-compatible API. A built-in web dashboard lets you manage configuration, sessions, memories, skills, and scheduled jobs from your browser.

## Architecture

Hermes runs in a single container:

- **Gateway** — the main process (`gateway run`). Handles chat platform connections, tool execution, memory, and scheduling. Exposes an OpenAI-compatible API on port 8642.
- **Dashboard** — the web UI on port 9119 runs as a background side-process inside the same container (enabled via `HERMES_DASHBOARD=1`). This is required because the dashboard's gateway-liveness detection relies on a shared PID namespace with the gateway process.

The container mounts your data directory at `/opt/data` and the Docker socket, enabling the **Docker terminal backend** — shell commands the agent runs are executed in fresh, isolated Docker containers with security hardening (capabilities dropped, no privilege escalation, PID limits).

## Before You Start

This app assumes you already have a working Hermes Agent data directory (the container mounts it at `/opt/data`). If you're setting up from scratch, the entrypoint bootstraps default config files — but you'll still need to configure your API keys and model provider.

Your data directory will be at the Runtipi app data path for hermes-agent under the `data/` subdirectory. It maps directly to what Hermes expects at `/opt/data` inside the container.

## Connecting to a Local Inference Server

If you run a local inference server (vLLM, Ollama, etc.) on your Runtipi host, configure the model in your data directory's `config.yaml`:

```yaml
model:
  provider: custom
  model: my-model
  base_url: http://host.docker.internal:8000/v1
  api_key: "none"
```

For Ollama:
```yaml
model:
  provider: custom
  model: llama3
  base_url: http://host.docker.internal:11434/v1
  api_key: "none"
```

See the [Docker guide](https://hermes-agent.nousresearch.com/docs/user-guide/docker#connecting-to-local-inference-servers) for more options (including Docker Compose setups with shared networks).

## Resources

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| Memory | 1 GB | 4 GB |
| CPU | 1 core | 2 cores |
| Disk (data volume) | 500 MB | 2+ GB |

Browser automation (Playwright/Chromium) is the most memory-hungry feature. Without it 1 GB is sufficient; with it allocate at least 2 GB.

## Links

- [Documentation](https://hermes-agent.nousresearch.com/docs)
- [Configuration reference](https://hermes-agent.nousresearch.com/docs/user-guide/configuration)
- [Docker guide](https://hermes-agent.nousresearch.com/docs/user-guide/docker)