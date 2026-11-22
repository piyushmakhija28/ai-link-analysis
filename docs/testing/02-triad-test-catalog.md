# Testing Architecture: Triad Test Cases Catalog

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  

---

## 1. Triad Test Cases Catalog (Positive, Negative, Edge-Case)

| Test ID | Test Category | Target Component | Description / Scenario | Expected Outcome |
|:---|:---|:---|:---|:---|
| **TC-01** | **Positive** | `app.main` (Bootstrap) | Application startup lifespan and health probe execution. | Uvicorn initializes, lifespan starts HTTPX pool, `/health` returns 200. |
| **TC-02** | **Positive** | `SSRFValidator` | Valid public HTTPS URL (`https://example.com`, `https://google.com`). | Passes validation; returns normalized URL string. |
| **TC-03** | **Negative** | `SSRFValidator` | Non-HTTP protocol (`file:///etc/passwd`, `gopher://127.0.0.1:6379`). | Raises `InvalidUrlSchemeException` (HTTP 400). |
| **TC-04** | **Negative** | `SSRFValidator` | Prohibited internal port (`https://example.com:22`, `:5432`, `:8080`). | Raises `ProhibitedPortException` (HTTP 400). |
| **TC-05** | **Edge-Case** | `SSRFValidator` | AWS/GCP cloud metadata IP (`http://169.254.169.254/latest/meta-data`). | Raises `SsrfViolationException` (HTTP 400). |
| **TC-06** | **Edge-Case** | `SSRFValidator` | Loopback IP variants (`127.0.0.1`, `127.0.0.2`, `::1`, `localhost`). | Raises `SsrfViolationException` (HTTP 400). |
| **TC-07** | **Edge-Case** | `SSRFValidator` | Private RFC 1918 subnets (`10.0.0.1`, `172.16.0.1`, `192.168.1.1`). | Raises `SsrfViolationException` (HTTP 400). |
| **TC-08** | **Positive** | `HtmlSemanticExtractor` | Standard HTML response with title, meta description, and headings. | Extracts clean pruned document with `<title>`, meta, headings, body. |
| **TC-09** | **Negative** | `HtmlSemanticExtractor` | Upstream web server returns HTTP 404 Not Found. | Raises `WebsiteUnreachableException` (HTTP 502). |
| **TC-10** | **Negative** | `HtmlSemanticExtractor` | Upstream web server returns HTTP 403 Forbidden / Cloudflare block. | Raises `WebsiteAccessDeniedException` (HTTP 502). |
| **TC-11** | **Negative** | `HtmlSemanticExtractor` | Upstream web server times out after 10.0s SLA. | Raises `WebsiteTimeoutException` (HTTP 504). |
| **TC-12** | **Edge-Case** | `HtmlSemanticExtractor` | Response payload exceeds 5MB streaming size limit. | Raises `WebsitePayloadTooLargeException` (HTTP 413). |
| **TC-13** | **Edge-Case** | `HtmlSemanticExtractor` | Web page with zero body or only whitespace. | Returns fallback metadata document without crashing. |
| **TC-14** | **Positive** | `GeminiClient` | Standard LLM response conforming to 5-section Pydantic schema. | Returns complete `LinkAnalysisResponse` with all 5 sections. |
| **TC-15** | **Negative** | `GeminiClient` | LLM returns malformed JSON with missing closing braces. | JSON repair loop recovers schema or falls back to offline mock. |
| **TC-16** | **Negative** | `GeminiClient` | Google GenAI API returns HTTP 429 Rate Limit. | Gracefully falls back to offline mock adapter, returning 200 with telemetry. |
| **TC-17** | **Edge-Case** | `GeminiClient` | Web page containing prompt injection payload. | XML fencing prevents instruction override; returns standard BI report. |
| **TC-18** | **Positive** | `OfflineMockClient` | Unconfigured `GEMINI_API_KEY` environment. | Generates deterministic high-fidelity BI report with `mock-diagnostic` model. |
| **TC-19** | **Positive** | `CacheService` | Second request to same normalized URL within 24 hours. | Serves response from cache in <5ms with `cache_hit: true` and $0.00 cost. |
| **TC-20** | **Positive** | `AnalysisRouter` | End-to-end `POST /link-analysis` with valid URL. | Returns HTTP 200 OK matching Dxora Section 4 JSON response structure. |
