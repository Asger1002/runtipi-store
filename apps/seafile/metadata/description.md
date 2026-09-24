# Seafile

Seafile Community Edition is a self-hosted file sync and sharing platform. It provides libraries, web access, file sharing links, file versioning, and desktop/mobile sync clients.

This app packages the official Seafile 13 Docker deployment for Runtipi with:

- `seafileltd/seafile-mc:13.0-latest`
- MariaDB 10.11
- Redis 7

## First start

During installation, choose strong values for:

- **Admin password**
- **Database root password**
- **Seafile database password**
- **JWT private key**

Generate a JWT private key with:

```sh
openssl rand -hex 32
```

The admin email/password are used only for the initial Seafile setup. Changing them later in Runtipi will not reset an existing Seafile admin account.

## Hostname and protocol

Set **Seafile hostname** to the hostname users will actually use, for example:

```text
seafile.catla-spica.ts.net
```

Set **Seafile protocol** to:

```text
https
```

when exposing it through Runtipi, Tailscale Serve, Cloudflare Tunnel, or another TLS reverse proxy.

## SeaDoc

This app currently disables SeaDoc (`ENABLE_SEADOC=false`) to keep the Runtipi integration simple and focused on evaluating Seafile file sync/sharing. SeaDoc can be added later as a separate routed service if needed.
