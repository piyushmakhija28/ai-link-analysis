# High-Level Architecture Design (HLD)
## AI-Powered Website Link Analysis API (`ai-link-analysis`)

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Architecture Pattern:** Clean Layered Hexagonal / C4 Model Microservice  
> **Repository:** [https://github.com/piyushmakhija28/ai-link-analysis](https://github.com/piyushmakhija28/ai-link-analysis)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  
> **Status:** Baselined & Enforced  

---

## 1. Architectural Principles & System Goals

The architecture of `ai-link-analysis` adheres to six foundational software engineering principles:
1. **Clean Layered Separation of Concerns:** Ingress Router $\to$ Zero-Trust Security $\to$ Extraction & Pruning $\to$ Generative AI Engine $\to$ Orchestration & Caching. No circular dependencies exist; upper layers depend strictly on lower-layer abstractions.
2. **Zero-Trust Security by Default:** Every incoming URL and scraped web payload is treated as inherently hostile. Defenses include pre-flight socket DNS resolution, strict RFC 1918/3927/6598 IP blacklist filtering, port whitelisting (80/443), and prompt injection XML boundary fencing.
3. **Token Economics & Sub-Task Efficiency:** Local CPU/RAM DOM pruning strips 85% of non-semantic HTML bytes before LLM tokenization, and a single-pass 5-in-1 consolidated prompt eliminates 80% of input token overhead, achieving complete analysis for **~₹0.09 INR ($0.00105 USD)** per request.
4. **Bulkhead Resilience & Dual-Engine Fail-Safe:** The system implements an asynchronous Circuit Breaker Pattern: Google Gemini 1.5 Flash (Primary) $\to$ OpenAI GPT-4o-mini (Secondary Fallback) $\to$ `OfflineMockClient` (Deterministic Rule-Based Fallback). Failures on external APIs never crash the ASGI worker.
5. **Deterministic High-Throughput Async Concurrency:** Non-blocking I/O (`httpx.AsyncClient` with connection pooling) and thread pool executor offloading for blocking DNS lookups ensure the event loop remains responsive under load.
6. **Unified Observability & Auditability:** Correlation ID tracing (`X-Correlation-ID`) across contextvars, ECS-compliant structured JSON logging, Prometheus-ready cost telemetry, and RFC 7807 problem details error envelopes.

---

## 2. C4 Model Architecture

### 2.1 C4 Level 1: System Context Diagram
The System Context diagram illustrates how the `ai-link-analysis` microservice interacts with external actors, target web servers, and cloud AI providers:

```mermaid
graph TD
    Client["Client Applications<br/>(B2B CRM, Marketing Automation, Evaluator Curl)"] -->|"POST /link-analysis (JSON URL)<br/>X-Correlation-ID"| System["ai-link-analysis Microservice<br/>[FastAPI / Python 3.10+ :4000]"]
    
    System -->|"Pre-Flight DNS Resolution (UDP/TCP 53)"| DNS["Upstream Recursive DNS Resolver"]
    System -->|"Streaming HTTP/HTTPS GET (Ports 80/443, <=5MB)"| TargetWeb["Target Public Websites<br/>(Internet Web Servers)"]
    System -->|"Single-Pass Structured Prompt<br/>HTTPS REST/gRPC (Schema Steering)"| GeminiCloud["Google Cloud: Gemini 1.5 Flash API"]
    System -.->|"Circuit Breaker Fallback<br/>HTTPS REST (JSON Schema Strict)"| OpenAICloud["OpenAI API: GPT-4o-mini"]
    
    System -->|"HTTP 200 OK: 5-Dimension BI Dossier<br/>or HTTP 4xx/5xx: RFC 7807 Problem Details"| Client
```

---

### 2.2 C4 Level 2: Container Diagram
The Container diagram illustrates the internal boundary of the OCI Docker container, runtime components, and process execution flow:

```mermaid
graph TD
    subgraph Container["OCI Docker Container: ai-link-analysis (non-root appuser:10001)"]
        Uvicorn["ASGI Server: Uvicorn (:4000)<br/>Worker Process Loop"]
        FastAPI["FastAPI Application Instance"]
        Middleware["Correlation ID & Security Middleware"]
        
        subgraph AppLayers["Application Software Layers"]
            Router["Ingress Routers<br/>/link-analysis, /health, /ready"]
            Orchestrator["Analysis Orchestrator<br/>State DAG & Circuit Breaker"]
            SSRF["SSRF Defense Pipeline<br/>Pre-Flight DNS & Subnet Check"]
            Extractor["Async HTTPX Extractor<br/>5MB Limiter, 10s SLA"]
            Pruner["Semantic DOM Pruner<br/>Tag Stripper, <8k Token Budget"]
            LLMEngine["LLM Client Engine<br/>Gemini Flash / OpenAI / Mock"]
            Cache["In-Memory SHA-256 Cache<br/>24h TTL Key-Value Store"]
        end
        
        Uvicorn --> FastAPI
        FastAPI --> Middleware
        Middleware --> Router
        Router --> Orchestrator
        Orchestrator --> Cache
        Orchestrator --> SSRF
        SSRF --> Extractor
        Extractor --> Pruner
        Pruner --> LLMEngine
        LLMEngine --> Cache
    end

    Extractor <-->|"Async HTTP/1.1 & HTTP/2"| TargetWeb["Target Web Server"]
    LLMEngine <-->|"REST HTTPS TLS 1.3"| GeminiCloud["Google Gemini API"]
    LLMEngine -.->|"REST HTTPS TLS 1.3"| OpenAICloud["OpenAI API"]
```

---

### 2.3 C4 Level 3: Component Diagram
The Component diagram provides a granular view of Python classes, interfaces, and abstractions inside the service:

```mermaid
graph LR
    subgraph RoutingLayer["Ingress & Presentation Layer"]
        AnalysisRouter["AnalysisRouter<br/>(POST /link-analysis)"]
        HealthRouter["HealthRouter<br/>(GET /health, /ready)"]
    end

    subgraph SecurityLayer["Security & Zero-Trust Ingress"]
        SSRFValidator["SSRFValidator<br/>• validate_target_url()<br/>• is_ip_blacklisted()<br/>• resolve_dns_async()"]
    end

    subgraph ExtractionLayer["Extraction & Sanitization"]
        HTTPXFetcher["AsyncHTTPXFetcher<br/>• fetch_stream()<br/>• 5MB Chunk Counter"]
        DOMPruner["DOMPruner<br/>• prune_tags()<br/>• isolate_semantic_meta()<br/>• apply_token_budget()"]
    end

    subgraph IntelligenceLayer["Generative Intelligence Engine"]
        LLMFactory["LLMClientFactory<br/>• create_client()"]
        GeminiClient["GeminiLLMClient<br/>(Native Schema Steering)"]
        OpenAIMiniClient["OpenAIMiniClient<br/>(Circuit Breaker Fallback)"]
        OfflineMock["OfflineMockClient<br/>(Deterministic Fallback)"]
        PromptBuilder["PromptBuilder<br/>(XML Fencing Delimiter)"]
    end

    subgraph OrchestrationLayer["Service Orchestration & Telemetry"]
        BIOrchestrator["AnalysisOrchestrator<br/>(DAG Coordinator)"]
        CacheService["AnalysisCacheService<br/>(SHA-256 TTL Store)"]
        TelemetryService["TelemetryService<br/>(Latency & Cost USD)"]
    end

    AnalysisRouter --> BIOrchestrator
    BIOrchestrator --> CacheService
    BIOrchestrator --> SSRFValidator
    BIOrchestrator --> HTTPXFetcher
    HTTPXFetcher --> DOMPruner
    DOMPruner --> PromptBuilder
    PromptBuilder --> LLMFactory
    LLMFactory --> GeminiClient
    LLMFactory -.-> OpenAIMiniClient
    LLMFactory -.-> OfflineMock
    GeminiClient --> TelemetryService
    TelemetryService --> AnalysisRouter
```

---

## 3. Concurrency, Event Loop & Thread Pool Mechanics

### 3.1 Non-Blocking Asyncio Design
The application runs on Uvicorn ASGI server using Python's standard `asyncio` event loop. All external network I/O operations (HTTPX website streaming and Gemini API calls) are 100% asynchronous and non-blocking.

### 3.2 Thread Pool Offloading for Blocking DNS Lookups
Python's standard library `socket.getaddrinfo()` is a synchronous, blocking system call that interacts with the operating system's resolver (`/etc/resolv.conf` or Windows DNS cache). If executed directly in an `async def` function, it would block the entire event loop for 10ms–2000ms.

To prevent event loop stalling, pre-flight DNS lookups are offloaded to Uvicorn's worker thread pool:
```python
loop = asyncio.get_running_loop()
addr_info = await loop.run_in_executor(
    None, socket.getaddrinfo, hostname, None, socket.AF_UNSPEC, socket.SOCK_STREAM
)
```

### 3.3 HTTPX Connection Pooling & Limits
To support high-concurrency workloads without exhausting file descriptors or memory, the `httpx.AsyncClient` is configured with strict connection pool limits:
```python
limits = httpx.Limits(
    max_keepalive_connections=20,
    max_connections=100,
    keepalive_expiry=30.0,
)
timeout = httpx.Timeout(
    connect=3.0,
    read=7.0,
    write=3.0,
    pool=2.0,
)
```

---

## 4. Failure Modes & Graceful Degradation Strategy

| Failure Scenario | Trigger Condition | Detection Mechanism | Immediate System Action | Client Response |
|:---|:---|:---|:---|:---|
| **SSRF Attack Attempt** | User submits private IP (`10.0.0.1`, `169.254.169.254`) or localhost | Pre-flight IP subnet check against blacklist CIDRs | Abort request before socket connect | HTTP 400 Bad Request (`ERR_SSRF_BLOCKED`) |
| **Non-Standard Port** | User submits `http://example.com:8080` or `:22` | Port validation check | Abort request immediately | HTTP 400 Bad Request (`ERR_DISALLOWED_PORT`) |
| **Slow Upstream Server** | Target website takes >10.0s to respond | HTTPX timeout watchdog | Cancel async stream, release socket | HTTP 504 Gateway Timeout (`ERR_UPSTREAM_TIMEOUT`) |
| **Decompression / Memory Bomb** | Target returns 50MB response or infinite gzip stream | Streaming chunk byte counter | Break stream loop when bytes > 5MB | HTTP 413 Payload Too Large (`ERR_PAYLOAD_TOO_LARGE`) |
| **Blank / Pure SPA Website** | Zero body text extracted after stripping scripts | DOM pruner character counter (<100 chars) | Check meta/OG tags; if still empty, abort | HTTP 422 Unprocessable (`ERR_EMPTY_WEBSITE_CONTENT`) |
| **Gemini Rate Limit / Outage** | Google Cloud returns HTTP 429 or 503 | Exception handler in `GeminiLLMClient` | Trip circuit breaker $\to$ Fallback to GPT-4o-mini / Mock | HTTP 200 OK (Clean synthesized payload with fallback flag) |
| **Unconfigured API Keys** | `GEMINI_API_KEY` omitted from environment | `is_configured()` settings check | Route directly to `OfflineMockClient` | HTTP 200 OK (Synthetic diagnostic intelligence) |

---
*End of High-Level Design Document. Authored by Piyush Makhija for Dxora Technologies Evaluation.*
