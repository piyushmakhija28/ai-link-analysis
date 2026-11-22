# Low-Level Design: Design Patterns Architecture

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  

---

## 1. Design Patterns Catalog

The microservice incorporates 8 enterprise design patterns to guarantee testability, maintainability, loose coupling, and resilience:

| Pattern | Component | Class / Implementation | Purpose & Benefit |
|:---|:---|:---|:---|
| **Strategy** | Content Extraction | `BaseContentExtractor`, `HtmlSemanticExtractor` | Enables swappable extraction algorithms without modifying the orchestrator. |
| **Factory Method** | AI Intelligence | `LLMClientFactory` | Instantiates `GeminiClient` or `OfflineMockClient` dynamically based on configuration. |
| **Adapter** | AI Provider | `GoogleGenAIAdapter` | Standardizes Google SDK outputs into internal domain Pydantic models. |
| **Pipeline / Filter** | Security Ingress | `SSRFValidator` filter chain | Sequentially evaluates scheme, host, DNS, IP subnet, port, and body size. |
| **Bulkhead** | Service Layer | `AnalysisOrchestrator` | Isolates scraping timeouts and LLM errors within bounded execution blocks. |
| **Dependency Injection** | FastAPI Routers | `FastAPI.Depends()` | Injects configurations, extractors, and services into controller handlers. |
| **Decorator** | Observability | `@timed_execution` | Cross-cutting latency, memory, and token telemetry tracking. |
| **Data Transfer Object** | Wire Boundaries | Pydantic v2 `BaseModel` | Strict external request/response schema validation and serialization. |

---

## 2. Pattern Deep-Dives

### 2.1 Factory Method Pattern: LLM Client Instantiation
```python
class LLMClientFactory:
    """Factory creating appropriate LLM client based on environment credentials."""

    @staticmethod
    def create_client(settings: Settings) -> BaseLLMClient:
        """Create a live GeminiClient if credentials exist, else OfflineMockClient.

        Args:
            settings: Loaded application settings.

        Returns:
            Configured implementation of BaseLLMClient.
        """
        if settings.is_configured():
            return GeminiClient(
                api_key=settings.gemini_api_key,
                model_name=settings.llm_model,
                timeout=settings.request_timeout,
            )
        return OfflineMockClient()
```

### 2.2 Strategy Pattern: Content Extraction
```python
class AnalysisOrchestrator:
    """Orchestrates link analysis by delegating to injected strategies."""

    def __init__(
        self,
        extractor: BaseContentExtractor,
        llm_client: BaseLLMClient,
        validator: SSRFValidator,
        cache: CacheService,
    ) -> None:
        self._extractor = extractor
        self._llm_client = llm_client
        self._validator = validator
        self._cache = cache
```
