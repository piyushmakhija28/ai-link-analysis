# Low-Level Design: Service Layer Contracts & Interface Protocols

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Repository:** [https://github.com/piyushmakhija28/ai-link-analysis](https://github.com/piyushmakhija28/ai-link-analysis)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  
> **Status:** Baselined & Enforced  

---

## 1. Architectural Philosophy

The service layer is the core coordination heart of the `ai-link-analysis` microservice. In alignment with **Clean Hexagonal Architecture**, all service boundaries are expressed via **Abstract Base Classes (ABCs)** and explicit **Python Protocols (`typing.Protocol`)**. 

Upper application layers (Routers and Controllers) depend exclusively on these abstract contracts, ensuring that concrete implementations (HTTPX, Google GenAI SDK, BeautifulSoup) can be seamlessly mocked, stubbed, or swapped during unit and integration testing without modifying core business logic.

---

## 2. Formal Service Layer Contracts

### 2.1 Content Extractor Protocol (`BaseWebsiteExtractor`)
```python
from abc import ABC, abstractmethod
from typing import Protocol, runtime_checkable
from app.schemas import PrunedDocument


@runtime_checkable
class WebsiteExtractorProtocol(Protocol):
    """Protocol defining the asynchronous contract for website streaming and pruning."""

    async def fetch_and_prune(
        self, url: str, correlation_id: str
    ) -> PrunedDocument:
        """Fetch upstream HTML, enforce 5MB streaming limit, prune DOM, and budget tokens.

        Pre-conditions:
            - Target URL has passed all 7 stages of SSRF validation.
            - Resolved IP is public and ports are 80 or 443.

        Post-conditions:
            - Returned PrunedDocument character count is <= 32,000 characters.
            - All active scripting elements (<script>, <style>) have been stripped.
            - High-signal metadata (<title>, meta descriptions) is extracted.

        Raises:
            UpstreamWebsiteTimeoutError: Fetch exceeded 10.0-second SLA.
            PayloadTooLargeError: Upstream body exceeded 5MB ceiling.
            EmptyWebsiteContentError: No extractable semantic text found.
            UpstreamWebsiteUnreachableError: DNS or HTTP 4xx/5xx network failure.
        """
        ...
```

---

### 2.2 LLM Synthesis Engine Protocol (`BaseLLMClient`)
```python
from abc import ABC, abstractmethod
from typing import Protocol, runtime_checkable
from app.schemas import LinkAnalysisPayload, PrunedDocument


@runtime_checkable
class LLMClientProtocol(Protocol):
    """Protocol defining the contract for structured business intelligence synthesis."""

    async def analyze(
        self, document: PrunedDocument, correlation_id: str
    ) -> LinkAnalysisPayload:
        """Execute single-pass multi-section prompt and synthesize 5 BI dimensions.

        Pre-conditions:
            - PrunedDocument contains valid semantic text (<8k tokens).
            - XML boundary fencing is applied around untrusted website content.

        Post-conditions:
            - Resulting LinkAnalysisPayload matches exact Pydantic schema structure.
            - Competitors list contains >= 3 verified entries.
            - Search queries cover all 5 required platforms (Google, FB, IG, LinkedIn, X).
            - Zero hallucinated numerical search volume fields exist in the payload.
            - SEO meta description length is <= 160 characters.

        Raises:
            LLMSynthesisError: Provider failed and no fallback client succeeded.
            SchemaValidationError: Generated JSON failed Pydantic schema validation.
        """
        ...

    def get_provider_name(self) -> str:
        """Return provider identifier (e.g. 'gemini-1.5-flash', 'gpt-4o-mini', 'offline-mock')."""
        ...
```

---

### 2.3 Cache Service Contract (`AnalysisCacheService`)
```python
from typing import Optional, Protocol, runtime_checkable
from app.schemas import LinkAnalysisResponse


@runtime_checkable
class CacheServiceProtocol(Protocol):
    """Protocol defining in-memory TTL caching for analysis responses."""

    def get(self, url: str) -> Optional[LinkAnalysisResponse]:
        """Retrieve pre-computed analysis response for a canonicalized URL.

        Returns:
            LinkAnalysisResponse if found and TTL is valid; None on cache miss or expiration.
        """
        ...

    def set(
        self,
        url: str,
        response: LinkAnalysisResponse,
        ttl_seconds: Optional[int] = None,
    ) -> None:
        """Store validated response under SHA-256(canonical_url) with expiration timestamp."""
        ...

    def clear(self) -> None:
        """Evict all entries from the cache."""
        ...
```

---

### 2.4 Master Orchestrator Contract (`AnalysisOrchestrator`)
```python
from app.schemas import LinkAnalysisRequest, LinkAnalysisResponse


class AnalysisOrchestrator:
    """Master workflow coordinator executing the Service Directed Acyclic Graph (DAG)."""

    def __init__(
        self,
        ssrf_validator: SSRFValidatorProtocol,
        extractor: WebsiteExtractorProtocol,
        primary_llm: LLMClientProtocol,
        fallback_llm: LLMClientProtocol,
        cache_service: CacheServiceProtocol,
    ) -> None:
        """Inject all required dependencies conforming to abstract protocols."""
        self._ssrf_validator = ssrf_validator
        self._extractor = extractor
        self._primary_llm = primary_llm
        self._fallback_llm = fallback_llm
        self._cache_service = cache_service

    async def analyze_link(
        self, request: LinkAnalysisRequest, correlation_id: str
    ) -> LinkAnalysisResponse:
        """Coordinate end-to-end execution: Cache Check -> SSRF -> Extractor -> LLM -> Cache Store -> Delivery."""
        ...
```

---
*End of Low-Level Design: Service Layer Contracts. Authored by Piyush Makhija for Dxora Technologies Evaluation.*
