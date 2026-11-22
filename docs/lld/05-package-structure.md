# Low-Level Design: Package Structure & Module Responsibilities

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  

---

## 1. Directory Layout & Module Responsibilities

```text
ai-link-analysis/
├── app/
│   ├── __init__.py                  # Application package initialization
│   ├── main.py                      # FastAPI lifespan, ASGI app, middleware, global exception handlers
│   ├── config.py                    # Pydantic Settings, environment variable loading
│   ├── constants.py                 # System constants, SSRF blacklist subnets, allowed ports
│   ├── exceptions.py                # Domain exception hierarchy, RFC 7807 problem details models
│   ├── logger.py                    # Centralized structured JSON logger with correlation ID
│   ├── schemas.py                   # Pydantic v2 request/response schemas for all 5 BI dimensions
│   │
│   ├── routers/
│   │   ├── __init__.py              # Routers package marker
│   │   ├── analysis.py              # POST /link-analysis endpoint handler
│   │   └── health.py                # SRE probes: GET /health (liveness), GET /ready (readiness)
│   │
│   ├── services/
│   │   ├── __init__.py              # Services package marker
│   │   ├── website_extractor.py     # Async HTTPX client, streaming 5MB cap, DOM pruning
│   │   ├── llm_service.py           # Gemini 1.5 Flash client, prompt builder, offline mock fallback
│   │   ├── analysis_orchestrator.py # Multi-stage business intelligence coordinator
│   │   └── cache_service.py         # SHA-256 URL hash TTL caching engine
│   │
│   └── utils/
│       ├── __init__.py              # Utilities package marker
│       └── ssrf_validator.py        # 7-stage zero-trust SSRF, DNS pre-flight & IP subnet validator
│
├── tests/
│   ├── __init__.py                  # Test package initialization
│   ├── conftest.py                  # Pytest fixtures, mock HTTP transports, mock LLM responses
│   ├── test_bootstrap.py            # App startup and lifespan lifecycle tests
│   ├── test_config.py               # Settings loading and validation tests
│   ├── test_ssrf_validator.py       # SSRF positive, negative, and edge-case attack tests
│   ├── test_website_extractor.py    # Scraping, timeout, 5MB cap, and DOM pruning tests
│   ├── test_llm_service.py          # Gemini client, schema repair, and offline mock fallback tests
│   ├── test_analysis_orchestrator.py# Bulkhead resilience and end-to-end orchestration tests
│   ├── test_router_analysis.py      # HTTP endpoint tests for POST /link-analysis
│   ├── test_router_health.py        # SRE health and readiness probe tests
│   └── test_e2e_flow.py             # Complete integration tests
│
├── docs/                            # 10-level enterprise documentation suite
├── uml/                             # 13 formal Mermaid UML diagrams
├── Dockerfile                       # Multi-stage non-root OCI container build
├── docker-compose.yml               # Local development container orchestration
├── render.yaml                      # Render.com Cloud Infrastructure-as-Code blueprint
├── openapi.yaml                     # Formal OpenAPI 3.1.0 specification
├── pyproject.toml                   # Project metadata, Ruff, Mypy, and Pytest configuration
├── requirements.txt                 # Frozen production dependencies
└── README.md                        # Master evaluation guide
```
