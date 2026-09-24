# Honcho

An AI-native cross-session memory engine by [Plastic Labs](https://plasticlabs.com). Honcho provides persistent user modeling with dialectic reasoning, session-scoped context injection, semantic search, and persistent conclusions.

It's the memory backend used by [Hermes Agent's Honcho memory provider plugin](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory-providers#1-honcho) and other AI tools.

## Architecture

This app runs four containers built from source:

| Service | Image | Purpose |
|---------|-------|---------|
| **honcho-api** | Built from source | FastAPI server — handles all Honcho API requests, database migrations |
| **honcho-deriver** | Built from source | Background worker — extracts observations, builds peer representations, generates summaries, runs dream consolidation |
| **honcho-db** | `pgvector/pgvector:pg15` | PostgreSQL with pgvector extension (vector embeddings) |
| **honcho-redis** | `redis:8.2` | Caching layer (enabled by default) |

The **honcho-api** service is the main entrypoint exposed on port 8000. The deriver runs in the background and is essential for Honcho's core functionality — without it, messages are stored but no memory or reasoning will occur.

## Initial Setup

### 1. First Install

The first install **builds from source** — Tipi will clone the Honcho repository and compile it. This takes **3–8 minutes** depending on your server's CPU and internet speed. Subsequent updates will only rebuild if the image is missing.

During install, Tipi will ask for your **LLM API Key**. This is required — the server will not start without an LLM provider configured.

### 2. Configure Your LLM Provider

By default, Honcho uses your API key with OpenAI (`gpt-5.4-mini` for text, `text-embedding-3-small` for embeddings).

**For OpenRouter / any OpenAI-compatible endpoint** (like OpenCode Go, vLLM, Ollama, LiteLLM):

Edit the `.env` file in your app data directory (`/path/to/runtipi/app-data/honcho/data/.env`) and add base URL overrides per feature:

```env
DERIVER_MODEL_CONFIG__OVERRIDES__BASE_URL=https://openrouter.ai/api/v1
DERIVER_MODEL_CONFIG__MODEL=google/gemini-2.5-flash

SUMMARY_MODEL_CONFIG__OVERRIDES__BASE_URL=https://openrouter.ai/api/v1
SUMMARY_MODEL_CONFIG__MODEL=google/gemini-2.5-flash
```

You need to set the override **for each feature module** that does LLM calls (Deriver, Dialectic, Summary, Dream, Embedding).

### 3. Verify

After install, check the logs to confirm everything started:

```bash
docker exec <prefix>-honcho-api-1 curl http://localhost:8000/health
```

Expected response: `{"status":"ok"}`

Run a smoke test:

```bash
curl -s -X POST http://localhost:8000/v3/workspaces \
  -H "Content-Type: application/json" \
  -d '{"name": "test"}'
```

## Connecting Hermes Agent to Honcho

Once Honcho is running, configure Hermes Agent to use it as its memory provider:

1. In your Hermes config, set:
```json
{
  "baseUrl": "http://your-server-ip:8000",
  "hosts": {
    "hermes": {
      "enabled": true,
      "aiPeer": "hermes",
      "peerName": "your-username",
      "workspace": "hermes"
    }
  }
}
```

2. Set `HONCHO_API_KEY` if you enabled auth, or leave empty for local-only access.

3. Run `hermes memory setup` in Hermes and select "honcho".

See the [Honcho Hermes integration docs](https://docs.honcho.dev/v3/guides/integrations/hermes) for full details.

## Authentication

By default, auth is disabled (`AUTH_USE_AUTH=false`). For production, enable auth by editing `.env`:

```env
AUTH_USE_AUTH=true
AUTH_JWT_SECRET=<generate with: docker exec <container> python scripts/generate_jwt_secret.py>
```

## Advanced Configuration

Every feature module can be independently configured with different models, base URLs, and parameters. See the [full .env.example](https://github.com/plastic-labs/honcho/blob/main/.env.template) in the Honcho repository for all available options. Edit the `.env` file in your app data directory and restart the app in Tipi.

Key configurable features:

- **Deriver** — Background memory worker (extracts observations, builds representations)
- **Dialectic** — Reasoning engine with 5 levels (minimal/low/medium/high/max)
- **Summary** — Session summarization (short: 20msgs/1000tok, long: 60msgs/4000tok)
- **Dream** — Deduction and induction (two-phase, idle timeout 60min)
- **Embedding** — Vector embedding model config
- **Peer Card** — User profile snapshots

## Resources

- [Honcho Documentation](https://docs.honcho.dev)
- [Self-hosting Guide](https://docs.honcho.dev/v3/contributing/self-hosting)
- [GitHub Repository](https://github.com/plastic-labs/honcho)
- [Hermes Agent Integration](https://docs.honcho.dev/v3/guides/integrations/hermes)
