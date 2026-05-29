FROM ghcr.io/astral-sh/uv:python3.11-bookworm-slim AS builder
WORKDIR /app

ENV UV_SYSTEM_PYTHON=1 \
    UV_COMPILE_BYTECODE=1 \
    UV_NO_PROGRESS=1

# Install deps first (cached layer), then copy code
COPY pyproject.toml .
# Install torch CPU-only explicitly BEFORE other packages to avoid CUDA variant
RUN uv pip install torch --index-url https://download.pytorch.org/whl/cpu
RUN uv pip install .

# ── Runtime stage ──────────────────────────────────────────────────────────────
FROM python:3.11-slim
WORKDIR /app

ENV PYTHONUNBUFFERED=1 \
    DOCKER_CONTAINER=1 \
    AWS_REGION=ap-south-1 \
    AWS_DEFAULT_REGION=ap-south-1

# Copy installed packages from builder
COPY --from=builder /usr/local/lib/python3.11 /usr/local/lib/python3.11
COPY --from=builder /usr/local/bin /usr/local/bin

# Install OTel separately (keep it lean)
RUN pip install --no-cache-dir aws-opentelemetry-distro==0.12.2

# Copy only what the agent needs
COPY 01_agentcore_runtime.py .
COPY lauki_qna.csv .

RUN useradd -m -u 1000 bedrock_agentcore
USER bedrock_agentcore

EXPOSE 8080

CMD ["opentelemetry-instrument", "python", "01_agentcore_runtime.py"]