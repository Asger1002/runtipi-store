# Watts On Data Reader

Collects heating and water history from the signed-in **Watts On** Android app once per day.

## How it works

- A persistent, privileged Android sidecar keeps the migrated Watts On app session.
- At the configured local time, the collector launches the app and captures a fresh short-lived bearer token through a private mitmproxy CA.
- It sends exactly one deliberate heating request and one deliberate water request, covering the two latest complete local calendar days.
- Parsing is offline. Available intervals are written to daily CSV files and idempotently upserted into SQLite.
- Only the newest contiguous zero-valued suffix is withheld as provider lag. Earlier zero-usage intervals remain valid. A heating interval is withheld only if all six heating measurements are zero.
- A failed attempt is recorded before execution and is not retried that day. The next day's two-day overlap catches up without duplicate rows.

## Runtipi settings

- **Daily snapshot time** — 24-hour `HH:MM`; defaults to `08:00`.
- **IANA time zone** — defaults to `Europe/Copenhagen`.
- **Heating and water device IDs** — stored as password-style Runtipi fields.

The app's status page reports only schedule and run state; it contains no device IDs, tokens, or measurements.

## Security and host requirements

This app is for the trusted `hs1` host. Its Android container is privileged because Android Binder is a kernel interface. The local `watts-android:16-ca-v1` and `watts-on-data-reader:0.2.0` images must be built on the host before installation. Android state and the mitmproxy CA private key live only in the app's private data directory and are never included in this app-store repository.

Do not expose ADB, mitmproxy, or the app externally. The compose file exposes only the sanitized status endpoint through Runtipi.
