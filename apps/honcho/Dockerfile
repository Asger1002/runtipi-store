# Honcho - Self-hosted LLM Memory Engine
# Builds from https://github.com/plastic-labs/honcho
#
# This Dockerfile clones the Honcho source repository and builds it.
# No pre-built image is available — the build happens at install time.

FROM python:3.13-slim-bookworm AS builder

# Install git for cloning the Honcho repository
RUN apt-get update && \
    apt-get install -y --no-install-recommends git && \
    rm -rf /var/lib/apt/lists/*

# Install uv (fast Python package manager)
COPY --from=ghcr.io/astral-sh/uv:0.9.24 /uv /bin/uv

# Clone Honcho source (shallow clone for speed)
WORKDIR /app
RUN git clone --depth 1 https://github.com/plastic-labs/honcho.git /tmp/honcho && \
    cp -r /tmp/honcho/. /app/ && \
    rm -rf /tmp/honcho

# Enable bytecode compilation
ENV UV_COMPILE_BYTECODE=1
ENV UV_LINK_MODE=copy
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Install project dependencies (excluding dev group)
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-install-project --no-group dev

# Sync the project
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-group dev

# Set runtime paths
ENV PATH="/app/.venv/bin:$PATH"
ENV HOME=/app
ENV UV_CACHE_DIR=/tmp/uv-cache

# Create non-root user
RUN addgroup --system app && \
    adduser --system --group app && \
    mkdir -p /tmp/uv-cache && \
    chown -R app:app /app /tmp/uv-cache

USER app

EXPOSE 8000
