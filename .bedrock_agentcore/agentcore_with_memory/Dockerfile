# ── Stage 1: Install dependencies ─────────────────────────────────────────────
FROM ghcr.io/astral-sh/uv:python3.11-bookworm-slim AS builder
WORKDIR /app

ENV UV_SYSTEM_PYTHON=1 \
    UV_COMPILE_BYTECODE=1 \
    UV_NO_PROGRESS=1

# Install torch CPU-only FIRST (avoids pulling 2GB CUDA variant)
RUN uv pip install torch --index-url https://download.pytorch.org/whl/cpu

# Install project dependencies
COPY pyproject.toml .
RUN uv pip install .

# ── Stage 2: Lean runtime image ────────────────────────────────────────────────
FROM python:3.11-slim
WORKDIR /app

ENV PYTHONUNBUFFERED=1 \
    DOCKER_CONTAINER=1 \
    AWS_REGION=ap-south-1 \
    AWS_DEFAULT_REGION=ap-south-1

# Copy installed packages from builder stage only
COPY --from=builder /usr/local/lib/python3.11 /usr/local/lib/python3.11
COPY --from=builder /usr/local/bin /usr/local/bin

# OTel — installed lean, no build tools needed
RUN pip install --no-cache-dir aws-opentelemetry-distro==0.12.2

# Copy only the files the agent actually needs at runtime
COPY 02_agentcore_memory.py .
COPY lauki_qna.csv .

RUN useradd -m -u 1000 bedrock_agentcore
USER bedrock_agentcore

EXPOSE 8080

CMD ["opentelemetry-instrument", "python", "02_agentcore_memory.py"]