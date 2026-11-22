# Testing Architecture: Mocking Strategy & Transport Isolation

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  

---

## 1. Hermetic Mocking Philosophy

To guarantee that tests run in any offline or restricted environment (such as air-gapped CI runners or candidate machines without active cloud billing), all external I/O boundaries are decoupled through transport-level and factory-level mocking:

1. **HTTPX MockTransport:** For web scraping tests, we mock at the HTTP transport layer using `httpx.MockTransport(handler)` rather than monkeypatching methods. This tests the actual HTTPX client logic, streaming chunk handlers, status code parsers, and header decoders.
2. **Google GenAI Client Mocking:** In unit tests for the Gemini client, `unittest.mock.AsyncMock` simulates `genai.Client.aio.models.generate_content`, allowing deterministic assertion of prompt formatting, schema steering, and exception handling.
3. **Offline Mock Fallback Adapter:** The factory pattern instantiates `OfflineMockClient` when credentials are absent, providing built-in zero-dependency execution.

---

## 2. HTTPX Mock Transport Fixture Example

```python
import httpx
import pytest


@pytest.fixture
def mock_html_transport():
    """MockTransport returning realistic website HTML."""

    def handler(request: httpx.Request) -> httpx.Response:
        if "timeout" in str(request.url):
            raise httpx.ReadTimeout("Request timed out", request=request)
        if "404" in str(request.url):
            return httpx.Response(404, request=request)
        if "huge" in str(request.url):
            return httpx.Response(200, content=b"A" * (6 * 1024 * 1024), request=request)
        return httpx.Response(
            200,
            text="<html><head><title>Acme Corp</title></head><body><h1>Services</h1></body></html>",
            request=request,
        )

    return httpx.MockTransport(handler)
```
