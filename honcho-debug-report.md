# Honcho Install Failure — Root Cause Found

## The Actual Error

```
validating docker-compose.generated.yml: services.honcho-api.networks.honcho_my-store_network
Additional property gw_priority is not allowed
```

Docker Compose exits with code 1 immediately on every install attempt because
the generated compose file uses `gw_priority`, a field only supported in
Docker Compose >= v2.35.0.

The server has Docker Compose v2.32.3 — too old by 3 minor versions.

## What Is gw_priority

Runtipi auto-injects this into every generated network block:

```yaml
networks:
  tipi_main_network:
    gw_priority: 1
  honcho_my-store_network:
    gw_priority: 0
```

It controls which network gets used as the default gateway for a container.
Runtipi added this in a recent release to control egress routing.
Older Docker Compose versions reject it as an unknown property.

This is NOT a problem with your store files.
config.json is valid. docker-compose.yml is valid.
The env-file path works (symlink is in place). The app.env exists.

## Fix Options

### Option A — Update Docker Compose on HS1 (recommended)

Install Docker Compose v2.35.0 or newer:

```bash
# Check latest release first:
curl -s https://api.github.com/repos/docker/compose/releases/latest | grep tag_name

# Download and replace the plugin binary:
COMPOSE_VERSION=v2.35.1   # replace with latest
curl -SL "https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-linux-x86_64" \
  -o /usr/local/lib/docker/cli-plugins/docker-compose
chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
docker compose version   # verify
```

Then retry the Honcho install from the Tipi UI — it should work immediately.

### Option B — Update Runtipi

If Runtipi itself is out of date, updating it may also bundle a newer
Docker Compose. Check current vs latest Runtipi version:

```bash
cat /root/runtipi/VERSION
curl -s https://api.github.com/repos/runtipi/runtipi/releases/latest | grep tag_name
```

### Option C — Patch Docker Compose to allow unknown properties (NOT recommended)

You could edit the JSON schema Docker Compose uses to validate compose files
to allow additional properties. This is fragile and breaks on updates.

## Summary

| Item         | Status |
|--------------|--------|
| config.json  | Valid  |
| docker-compose.yml | Valid |
| app.env path | Works (symlink in place) |
| LLM_OPENAI_API_KEY | Set correctly in app.env |
| Docker Compose version | v2.32.3 — too old (need >= v2.35.0) |

**Go with Option A.** Update Docker Compose, then install Honcho from the UI.
