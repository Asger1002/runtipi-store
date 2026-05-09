# Honcho - Self-hosted LLM Memory Engine
# Builds from https://github.com/plastic-labs/honcho
#
# Based on upstream Dockerfile.example, adapted for runtipi:
#   - Clones the source at build time (no pre-existing source in context)
#   - Includes workaround for uv build-standalone Python prefix
#
# References:
#   https://pythonspeed.com/articles/base-image-python-docker-images/
#   https://testdriven.io/blog/docker-best-practices/

FROM python:3.13-slim-bookworm

# Install git for cloning the Honcho repository
RUN apt-get update && \
    apt-get install -y --no-install-recommends git && \
    rm -rf /var/lib/apt/lists/*

# Install uv (fast Python package manager)
COPY --from=ghcr.io/astral-sh/uv:0.9.24 /uv /bin/uv

WORKDIR /app

# ── Stage 1: Clone dependency metadata (for Docker layer caching) ─────
# Clone shallow, extract only lockfile + project config first so that
# dependency installation can be cached independently of source changes.
RUN git clone --depth 1 https://github.com/plastic-labs/honcho.git /tmp/honcho && \
    cp /tmp/honcho/uv.lock /tmp/honcho/pyproject.toml /app/

# Enable bytecode compilation
ENV UV_COMPILE_BYTECODE=1

# Copy from the cache instead of linking since it's a mounted volume
ENV UV_LINK_MODE=copy

# Python optimizations
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Install uv-managed Python to a world-accessible location.
# uv downloads Python build-standalone interpreters that have the
# install prefix baked in as /install. Storing them under /opt/uv/
# ensures the app user can access stdlib files at runtime.
ENV UV_PYTHON_INSTALL_DIR=/opt/uv/python

# Install project dependencies (excluding the project itself, for caching)
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-install-project --no-group dev

# ── Stage 2: Copy full source and complete installation ───────────────
# Copy the remaining source files from the cloned repo
RUN cp -r /tmp/honcho/. /app/ && rm -rf /tmp/honcho

# Sync the project (installs the app package itself)
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-group dev

# uv-downloaded Python interpreters have /install baked in as their prefix
# (build-standalone convention). Create a symlink so Python can find its
# standard library at runtime under the expected path.
RUN ln -sfn "$(dirname "$(dirname "$(readlink -f /app/.venv/bin/python)")")" /install

# Place executables in the environment at the front of the path
ENV PATH="/app/.venv/bin:$PATH"
ENV HOME=/app
ENV UV_CACHE_DIR=/tmp/uv-cache

# Create non-root user and set ownership
RUN addgroup --system app && \
    adduser --system --group app && \
    mkdir -p /tmp/uv-cache && \
    chown -R app:app /app /tmp/uv-cache

# Switch to non-root user
USER app

EXPOSE 8000

# Default command (overridden by docker-compose entrypoint for each service)
CMD ["fastapi", "run", "--host", "0.0.0.0", "src/main.py"]
