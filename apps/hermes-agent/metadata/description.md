# Hermes Agent

A self-hosted AI agent by [Nous Research](https://nousresearch.com). Hermes runs as a persistent gateway that connects to your preferred chat platform (Telegram, Discord, Slack, WhatsApp) and/or exposes an OpenAI-compatible API. A built-in web dashboard lets you manage configuration, sessions, memories, skills, and scheduled jobs from your browser.

## Architecture

This app runs two containers:

- **hermes** — the gateway process. Handles chat platform connections, tool execution, memory, and scheduling. Exposes an OpenAI-compatible API on port 8642 (internal only).
- **hermes-dashboard** — the web UI on port 9119. Read-only access to the data directory; detects gateway health via `http://hermes:8642`.

Both containers share the same data volume (`/opt/data`) and the Docker socket, enabling the **Docker terminal backend** — shell commands the agent runs are executed in fresh, isolated Docker containers with security hardening (capabilities dropped, no privilege escalation, PID limits). This is important for safe code execution on a server.

## Initial Setup

On first install, the data volume is bootstrapped automatically (default `config.yaml`, `.env.example` → `.env`). Before the gateway can connect to any AI provider or chat platform, you need to configure your API keys.

**Option 1 — via Runtipi terminal / SSH into the host:**
```bash
docker exec -it <hermes-container-name> /opt/hermes/.venv/bin/hermes config set OPENAI_API_KEY sk-...
# or
docker exec -it <hermes-container-name> /opt/hermes/.venv/bin/hermes config set ANTHROPIC_API_KEY sk-ant-...
```

**Option 2 — edit the .env file directly:**
```bash
# On the host, edit the data directory's .env:
nano /path/to/runtipi/app-data/hermes-agent/data/.env
```

**Option 3 — run the interactive setup wizard:**
```bash
docker run -it --rm \
  -v /path/to/runtipi/app-data/hermes-agent/data:/opt/data \
  nousresearch/hermes-agent setup
```

After configuring keys, restart the app in Runtipi to apply.

## Docker Terminal Backend

The `/var/run/docker.sock` is mounted into both containers so Hermes can use the **Docker terminal backend** for sandboxed code execution. To activate it, set in your config:

```bash
docker exec -it <hermes-container-name> /opt/hermes/.venv/bin/hermes config set terminal.backend docker
```

Or edit `config.yaml` in the data directory:
```yaml
terminal:
  backend: docker
  docker_image: "nikolaik/python-nodejs:python3.11-nodejs20"
  container_persistent: true
```

## Resources

- Memory: 4 GB (gateway) + 512 MB (dashboard) recommended
- CPU: 2 cores (gateway) + 0.5 cores (dashboard)
- Disk: 2+ GB for data volume (grows with sessions, skills, and memories)

## Links

- [Documentation](https://hermes-agent.nousresearch.com/docs)
- [Configuration reference](https://hermes-agent.nousresearch.com/docs/user-guide/configuration)
- [Docker guide](https://hermes-agent.nousresearch.com/docs/user-guide/docker)
