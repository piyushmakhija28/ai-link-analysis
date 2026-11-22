# UML 03: C4 Component Diagram

```mermaid
C4Component
    title Component Diagram for ai-link-analysis Application Core

    Container_Boundary(api_app, "FastAPI Microservice Application")
        Component(router_analysis, "AnalysisRouter", "app/routers/analysis.py", "Handles POST /link-analysis, parses request body, injects correlation ID")
        Component(router_health, "HealthRouter", "app/routers/health.py", "Exposes /health (liveness) and /ready (readiness) SRE probes")
        Component(orchestrator, "AnalysisOrchestrator", "app/services/analysis_orchestrator.py", "Coordinates DAG execution, bulkhead isolation, and circuit breaker")
        Component(cache_service, "AnalysisCacheService", "app/services/cache_service.py", "SHA-256 URL hashing and in-memory TTL expiration store")
        Component(ssrf_validator, "SSRFValidator", "app/utils/ssrf_validator.py", "7-stage zero-trust scheme, port, DNS, and IP subnet inspection")
        Component(extractor, "AsyncHTTPXWebsiteExtractor", "app/services/website_extractor.py", "Streams HTML chunks, enforces 5MB ceiling and 10s SLA")
        Component(pruner, "DOMPruner", "app/services/website_extractor.py", "Strips active scripts/styles and caps text context to <32k chars")
        Component(prompt_builder, "PromptBuilder", "app/services/llm_service.py", "Constructs XML-fenced single-pass prompt with negative constraints")
        Component(llm_factory, "LLMClientFactory", "app/services/llm_service.py", "Instantiates Gemini, OpenAI, or Mock clients dynamically")
        Component(gemini_client, "GeminiLLMClient", "app/services/llm_service.py", "Google GenAI SDK client with native response_schema steering")
        Component(openai_client, "OpenAIMiniLLMClient", "app/services/llm_service.py", "OpenAI client fallback with strict json_schema enforcement")
        Component(mock_client, "OfflineMockClient", "app/services/llm_service.py", "Deterministic synthetic intelligence for CI and offline modes")
        Component(circuit_breaker, "CircuitBreaker", "app/services/llm_service.py", "Fault-tolerance state machine managing failover")
        Component(telemetry, "TelemetryService", "app/utils/token_counter.py", "Calculates latency, token counts, and USD/INR costs")
    Container_Boundary_End

    Rel(router_analysis, orchestrator, "Delegates analysis task")
    Rel(orchestrator, cache_service, "Checks and sets cache")
    Rel(orchestrator, ssrf_validator, "Validates target URL")
    Rel(orchestrator, extractor, "Fetches target website")
    Rel(extractor, pruner, "Prunes raw HTML")
    Rel(pruner, prompt_builder, "Supplies cleaned PrunedDocument")
    Rel(orchestrator, circuit_breaker, "Wraps LLM execution")
    Rel(circuit_breaker, llm_factory, "Requests client strategy")
    Rel(llm_factory, gemini_client, "Primary client")
    Rel(llm_factory, openai_client, "Fallback client")
    Rel(llm_factory, mock_client, "Offline client")
    Rel(orchestrator, telemetry, "Assembles execution metrics")
    Rel(telemetry, router_analysis, "Delivers enriched LinkAnalysisResponse")
```
