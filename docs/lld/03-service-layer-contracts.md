# Low-Level Design: Service Layer Contracts

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  

---

## 1. Analysis Orchestrator Service Contract

### 1.1 Method: `analyze_link`
```python
async def analyze_link(
    self,
    url: str,
    correlation_id: str,
    force_refresh: bool = False,
) -> LinkAnalysisResponse:
    """Execute end-to-end link analysis workflow.

    1. Checks cache unless force_refresh is True.
    2. Validates URL against SSRF and network blacklist.
    3. Fetches and prunes HTML to under 8,000 tokens.
    4. Invokes LLM service to extract 5-section business intelligence.
    5. Saves result in cache with 24h TTL.
    6. Enriches response with telemetry and correlation ID.

    Args:
        url: Validated public website URL string.
        correlation_id: Distributed tracing UUID.
        force_refresh: Bypass cache if True.

    Returns:
        Complete LinkAnalysisResponse matching Dxora Section 4 contract.

    Raises:
        SsrfViolationException: On private/internal IP destination.
        WebsiteExtractionException: On inaccessible website.
        LLMServiceException: On intelligence synthesis failure.
    """
```

---

## 2. SSRF Validator Contract

### 2.1 Method: `validate_url`
```python
def validate_url(self, url: str) -> str:
    """Validate URL syntax, protocol, DNS resolution, and IP safety.

    Args:
        url: User-provided URL.

    Returns:
        Normalized safe URL string.

    Raises:
        InvalidUrlSchemeException: If scheme is not http or https.
        ProhibitedPortException: If port is not 80 or 443.
        SsrfViolationException: If target resolves to private/loopback IP.
    """
```

---

## 3. Cache Service Contract

### 3.1 Methods: `get` and `set`
```python
def get(self, url: str) -> Optional[LinkAnalysisResponse]:
    """Retrieve cached analysis if present and within TTL."""


def set(self, url: str, data: LinkAnalysisResponse, ttl: int = 86400) -> None:
    """Store analysis result under SHA-256(normalized_url) key."""
```
