# Nextcloud (All-in-One) — Nextcloud 35 + Collabora

Self-hosted file sync and share, with **Collabora Online built in** (LibreOffice-based) so documents, spreadsheets and presentations are editable directly in the browser.

## Access (Tailscale-only)

This app is designed to be reached **only through your tailnet** — there is no public internet exposure:

- **Browser**: `https://nextcloud.catla-spica.ts.net` via Tailscale Serve (valid Let's Encrypt cert, no Cloudflare, no public DNS). Note: Tailscale Services use the tailnet root domain — no `hs1.` prefix.
- Nextcloud trusts this hostname (`NEXTCLOUD_TRUSTED_DOMAINS`) and is told the scheme is HTTPS (`OVERWRITEPROTOCOL=https`), so CalDAV/CardDAV, desktop and mobile clients all work.

## What runs

| Container | Purpose |
|---|---|
| `nextcloud` (nextcloud:35.0.0-apache) | Main app |
| `db-nextcloud` (postgres:17) | Database |
| `redis-nextcloud` (redis:7.4.11) | Caching / transactions |
| `cron` | Background jobs equivalent (`/cron.sh`) |
| `collabora` (collabora/code:25.04.4.2.1) | WOPI office editing server, talks to Nextcloud over the internal Docker network only |

## Office editing

Collabora is pre-wired: `richdocuments` can be enabled in Nextcloud's app store once logged in, and Nextcloud automatically discovers the WOPI endpoint on the internal network. Only the Tailscale HTTPS hostname is allowed as WOPI callback, so no admin URL fiddling is needed.

## Notes

- Newest stable Nextcloud release line (35.0.0) — newer than the official Runtipi store's 34.0.4 pin.
- Not based on Nextcloud's official AIO image; instead the same "all-in-one" outcome is achieved with a single compose project bundling Nextcloud + Collabora, avoiding the AIO image's built-in Caddy/TLS which doesn't fit behind Runtipi's Traefik.
