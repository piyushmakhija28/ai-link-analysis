# Low-Level Design: Class Architecture & Object Contracts

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  

---

## 1. Core Class Diagram & Structure

```mermaid
classDiagram
    class Settings {
        +str app_name
        +str app_env
        +int port
        +str gemini_api_key
        +str llm_model
        +int max_content_length
        +int request_timeout
        +int cache_ttl
        +list allowed_ports
        +bool is_configured() bool
    }

    class SSRFValidator {
        +validate_url(url: str) ValidatedUrl
        -_check_scheme(parsed) void
        -_check_hostname(host: str) void
        -_resolve_dns_and_filter(host: str) list[str]
        -_check_port(port: int) void
    }

    class BaseContentExtractor {
        <<interface>>
        +extract(url: str) ExtractedContent*
    }

    class HtmlSemanticExtractor {
        -httpx.AsyncClient client
        -DOMPruner pruner
        +extract(url: str) ExtractedContent
        -_fetch_stream(url: str) str
    }

    class DOMPruner {
        +prune_and_budget(raw_html: str, max_chars: int) PrunedDocument
        -_strip_noise(soup) void
        -_extract_semantic_meta(soup) dict
        -_extract_headings_and_body(soup) str
    }

    class BaseLLMClient {
        <<interface>>
        +analyze(content: PrunedDocument) AnalysisResult*
    }

    class GeminiClient {
        -genai.Client client
        -str model_name
        +analyze(content: PrunedDocument) AnalysisResult
        -_build_prompt(content: PrunedDocument) str
    }

    class OfflineMockClient {
        +analyze(content: PrunedDocument) AnalysisResult
        -_synthesize_competitors(meta) list
        -_synthesize_queries(meta) list
        -_synthesize_seo(meta) dict
        -_synthesize_services(meta) list
        -_synthesize_audience(meta) dict
    }

    class LLMClientFactory {
        +create_client(settings: Settings) BaseLLMClient$
    }

    class AnalysisOrchestrator {
        -SSRFValidator ssrf_validator
        -BaseContentExtractor extractor
        -BaseLLMClient llm_client
        -CacheService cache_service
        +analyze_link(url: str, correlation_id: str) LinkAnalysisResponse
    }

    class CacheService {
        -dict _memory_store
        +get(url: str) Optional[LinkAnalysisResponse]
        +set(url: str, data: LinkAnalysisResponse, ttl: int) void
        -_hash_url(url: str) str
    }

    BaseContentExtractor <|.. HtmlSemanticExtractor
    HtmlSemanticExtractor --> DOMPruner
    BaseLLMClient <|.. GeminiClient
    BaseLLMClient <|.. OfflineMockClient
    LLMClientFactory ..> BaseLLMClient
    AnalysisOrchestrator --> SSRFValidator
    AnalysisOrchestrator --> BaseContentExtractor
    AnalysisOrchestrator --> BaseLLMClient
    AnalysisOrchestrator --> CacheService
```

---

## 2. Key Interface Signatures

### 2.1 Content Extractor Interface
```python
from abc import ABC, abstractmethod
from app.schemas import PrunedDocument


class BaseContentExtractor(ABC):
    """Abstract interface defining the contract for web content extraction."""

    @abstractmethod
    async def extract(self, url: str) -> PrunedDocument:
        """Asynchronously extract and prune semantic content from target URL.

        Args:
            url: Validated public website URL.

        Returns:
            PrunedDocument containing title, meta, headings, and bounded body text.

        Raises:
            WebsiteExtractionException: On HTTP/network failure.
        """
        pass
```

### 2.2 LLM Client Interface
```python
from abc import ABC, abstractmethod
from app.schemas import AnalysisResult, PrunedDocument


class BaseLLMClient(ABC):
    """Abstract interface defining the contract for business intelligence extraction."""

    @abstractmethod
    async def analyze(self, content: PrunedDocument) -> AnalysisResult:
        """Analyze pruned website document and extract 5 structured BI sections.

        Args:
            content: Pruned semantic website text and metadata.

        Returns:
            AnalysisResult with all 5 verified BI sections.

        Raises:
            LLMServiceException: On API rate-limit, auth, or schema parsing error.
        """
        pass
```
