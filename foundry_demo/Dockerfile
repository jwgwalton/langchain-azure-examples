FROM python:3.12-slim

# Copy the standalone uv binaries from the official image instead of installing
# uv and its build dependencies in the final image.
COPY --from=ghcr.io/astral-sh/uv:0.8.17 /uv /uvx /bin/

# Use a fixed, unprivileged UID and a writable app directory so a compromised
# process does not run as root and file ownership is predictable at runtime.
RUN groupadd --gid 10001 app && \
	useradd --uid 10001 --gid app --create-home --shell /usr/sbin/nologin app && \
	install -d --owner app --group app /app

WORKDIR /app

# Install dependencies in a separate layer so it remains cached until the
# project metadata or lockfile changes.
COPY --chown=app:app pyproject.toml uv.lock ./
USER app
RUN uv sync --locked --no-dev --no-install-project --no-cache

# Copy application code afterward so source changes do not reinstall dependencies.
COPY --chown=app:app main.py ./

# Foundry can override PORT at runtime; EXPOSE documents the local default only.
ENV PATH="/app/.venv/bin:$PATH" \
	PORT=8088 \
	PYTHONDONTWRITEBYTECODE=1 \
	PYTHONUNBUFFERED=1

EXPOSE 8088

CMD ["python", "main.py"]
