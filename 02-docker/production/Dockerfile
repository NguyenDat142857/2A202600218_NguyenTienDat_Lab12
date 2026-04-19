# ============================================================
# ADVANCED Dockerfile — Multi-stage build (FIXED)
# ============================================================

# ──────────────────────────────────────────────────────────
# STAGE 1: Builder
# ──────────────────────────────────────────────────────────
FROM python:3.11-slim AS builder

WORKDIR /app

# Install build deps
RUN apt-get update && apt-get install -y \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements từ ROOT context
COPY 02-docker/production/requirements.txt .

# Install deps vào /root/.local
RUN pip install --no-cache-dir --user -r requirements.txt


# ──────────────────────────────────────────────────────────
# STAGE 2: Runtime
# ──────────────────────────────────────────────────────────
FROM python:3.11-slim AS runtime

# Tạo non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /app

# Copy packages từ builder
COPY --from=builder /root/.local /home/appuser/.local

# Copy source code từ ROOT context
COPY 02-docker/production/main.py .

# Copy utils từ ROOT
RUN mkdir -p /app/utils
COPY utils/mock_llm.py /app/utils/mock_llm.py

# Set quyền
RUN chown -R appuser:appuser /app

# Switch user
USER appuser

# Env
ENV PATH=/home/appuser/.local/bin:$PATH
ENV PYTHONPATH=/app

# Expose
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')" || exit 1

# Run
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "2"]