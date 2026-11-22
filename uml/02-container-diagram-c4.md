# UML 02: C4 Container Diagram

```mermaid
C4Container
    title Container Architecture Diagram for ai-link-analysis

    Person(user, "API Consumer", "Client submitting website URLs for business intelligence analysis")

    Container_Boundary(c1, "Docker Container: ai-link-analysis (non-root appuser:10001)")
        Component(uvicorn, "Uvicorn ASGI Server", "Python 3.13 / Uvicorn 0.34", "High-performance non-blocking event loop worker process")
        Component(fastapi, "FastAPI Web Application", "FastAPI 0.115+", "Routing, OpenAPI 3.1 generation, and RFC 7807 error handling")
        Component(middleware, "Correlation & Security Middleware", "Python contextvars", "X-Correlation-ID injection and request duration tracing")
        Component(orchestrator, "Analysis Orchestrator", "Python Service DAG", "Coordinates cache checks, SSRF, extraction, and dual-engine LLM execution")
        Component(ssrf, "SSRF Defense Pipeline", "Python ipaddress & socket", "7-stage validation blocking private subnets, cloud metadata, and bad ports")
        Component(extractor, "Async HTTPX Extractor", "HTTPX 0.28+", "Streams HTML with 5MB ceiling and 10.0s cumulative timeout")
        Component(pruner, "DOM Pruner & Budgeter", "BeautifulSoup4 / lxml", "Strips active scripts/styles and caps text to <32k chars (<8k tokens)")
        Component(llm_engine, "LLM Synthesis Engine", "Google GenAI & OpenAI SDKs", "Gemini 1.5 Flash client with circuit breaker fallback to GPT-4o-mini and Mock")
        Component(cache, "In-Memory SHA-256 Cache", "Python Dict with TTL", "Caches responses for 24 hours serving repeat queries in <5ms at $0.00")
    Container_Boundary_End

    System_Ext(dns, "Public DNS Resolver", "Resolves hostnames to IPv4/IPv6 addresses")
    System_Ext(web, "Target Web Server", "Public internet website")
    System_Ext(gemini, "Google Gemini Cloud API", "Gemini 1.5 Flash model")
    System_Ext(openai, "OpenAI API", "GPT-4o-mini model")

    Rel(user, uvicorn, "POST /link-analysis", "HTTPS")
    Rel(uvicorn, fastapi, "Dispatches ASGI request")
    Rel(fastapi, middleware, "Passes request")
    Rel(middleware, orchestrator, "Executes workflow")
    Rel(orchestrator, cache, "Checks SHA-256 cache")
    Rel(orchestrator, ssrf, "Validates URL safety")
    Rel(ssrf, dns, "Resolves hostname asynchronously", "UDP 53")
    Rel(orchestrator, extractor, "Initiates stream fetch")
    Rel(extractor, web, "GET target URL (<=5MB)", "HTTP/HTTPS")
    Rel(extractor, pruner, "Passes raw HTML bytes")
    Rel(pruner, orchestrator, "Returns PrunedDocument (<8k tokens)")
    Rel(orchestrator, llm_engine, "Synthesizes 5 dimensions")
    Rel(llm_engine, gemini, "Primary structured query", "HTTPS REST")
    Rel(llm_engine, openai, "Circuit breaker fallback", "HTTPS REST")
    Rel(orchestrator, user, "HTTP 200 OK JSON Response", "JSON")
```
