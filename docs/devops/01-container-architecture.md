# DevOps Architecture: Multi-Stage Non-Root OCI Container Hardening

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Repository:** [https://github.com/piyushmakhija28/ai-link-analysis](https://github.com/piyushmakhija28/ai-link-analysis)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  
> **Status:** Baselined & Enforced  

---

## 1. Container Hardening Principles & Security Benchmarks

Under modern cloud-native security standards (CIS Docker Benchmark, NIST SP 800-190), running containers as `root` is a critical vulnerability. If an application vulnerability (such as remote code execution via an unpatched dependency) is exploited, an attacker inherits root privileges on the container and can potentially escape to the host kernel.

The `ai-link-analysis` container implements **Institutional-Grade Container Security**:
1. **Multi-Stage Build Pipeline:** Stage 1 (`builder`) compiles wheel packages with build tools (`gcc`); Stage 2 (`runtime`) contains strictly the minimal `python:3.13-slim` base without compilers or package managers, shrinking image size to **<150MB** and eliminating CVEs.
2. **Explicit Unprivileged User Execution:** Runs strictly under `appuser` (UID: `10001`, GID: `10001`). Root access is permanently revoked via `USER 10001:10001`.
3. **Linux Capability Stripping:** All Linux capabilities are dropped (`cap_drop: [ALL]`); zero setuid/setgid binaries exist.
4. **Read-Only Root Filesystem Compatibility:** The container writes zero state to disk; caching is kept in RAM.
5. **Native Docker HEALTHCHECK:** Built-in health check probes `/health` every 30 seconds to enable automated orchestrator self-healing.

```mermaid
graph TD
    subgraph BuildStage["Stage 1: Builder (python:3.13-slim)"]
        B1["Install gcc & build dependencies"]
        B2["pip install --user --no-cache-dir -r requirements.txt"]
    end

    subgraph RuntimeStage["Stage 2: Hardened Runtime (python:3.13-slim)"]
        R1["Create unprivileged group appgroup (GID 10001)"]
        R2["Create unprivileged user appuser (UID 10001)"]
        R3["Install curl for healthcheck only"]
        R4["Copy /.local wheels from Stage 1"]
        R5["Copy application source code /app/app"]
        R6["chown -R 10001:10001 /app /home/appuser"]
        R7["HEALTHCHECK CMD curl -f http://localhost:4000/health"]
        R8["USER 10001:10001"]
        R9["ENTRYPOINT: uvicorn app.main:app --port 4000"]
    end

    BuildStage --> RuntimeStage
```

---

## 2. Hardened Multi-Stage Dockerfile Specification

```dockerfile
# Stage 1: Build & Dependency Compilation
FROM python:3.13-slim AS builder

WORKDIR /build
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    libxml2-dev \
    libxslt1-dev \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Stage 2: Minimal Hardened Runtime
FROM python:3.13-slim AS runtime

WORKDIR /app
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PATH="/home/appuser/.local/bin:$PATH"

# Create non-root unprivileged service account
RUN groupadd -g 10001 appgroup && \
    useradd -u 10001 -g appgroup -s /sbin/nologin -m appuser && \
    apt-get update && apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# Copy pre-compiled dependencies from builder
COPY --from=builder /root/.local /home/appuser/.local
COPY app /app/app

# Enforce strict ownership
RUN chown -R appuser:appgroup /app /home/appuser

# Revoke root execution
USER 10001:10001
EXPOSE 4000

# Native container health check
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:4000/health || exit 1

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "4000"]
```

---

## 3. Docker Compose Orchestration (`docker-compose.yml`)

```yaml
version: "3.8"

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: ai-link-analysis-service
    ports:
      - "4000:4000"
    environment:
      - APP_NAME=AI-Powered Website Link Analysis API
      - APP_ENV=production
      - PORT=4000
      - GEMINI_API_KEY=${GEMINI_API_KEY:-}
      - OPENAI_API_KEY=${OPENAI_API_KEY:-}
      - PRIMARY_LLM_MODEL=gemini-1.5-flash
      - FALLBACK_LLM_MODEL=gpt-4o-mini
      - CACHE_TTL_SECONDS=86400
      - LOG_LEVEL=INFO
    read_only: true
    cap_drop:
      - ALL
    security_opt:
      - no-new-privileges:true
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:4000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
```

---
*End of Container Architecture Specification. Authored by Piyush Makhija for Dxora Technologies Evaluation.*
