# Testing Architecture: Exhaustive Triad Test Cases Catalog (35+ Scenarios)

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Repository:** [https://github.com/piyushmakhija28/ai-link-analysis](https://github.com/piyushmakhija28/ai-link-analysis)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  
> **Status:** Baselined & Enforced  

---

## 1. Complete Triad Test Cases Catalog

The following catalog defines the 35+ test cases implemented across the pytest triad suite:

| Test ID | Test Category | Target Component | Test Function Identifier | Scenario & Ingress Payload | Expected Assertions & HTTP Status |
|:---:|:---|:---|:---|:---|:---|
| **TC-01** | **Positive** | `app.config` | `test_settings_load_defaults` | Instantiates `Settings()` with default environment values. | Port 4000, environment 'production', cache TTL 86400s. |
| **TC-02** | **Positive** | `app.schemas` | `test_request_schema_valid_url` | `{"url": "https://example.com"}` | Pydantic model parses cleanly; URL normalized. |
| **TC-03** | **Negative** | `app.schemas` | `test_request_schema_invalid_url` | `{"url": "not-a-valid-url"}` | Raises `ValidationError` on malformed URI syntax. |
| **TC-04** | **Positive** | `SSRFValidator` | `test_ssrf_allows_public_https_url` | `https://dxora.com` (Resolves to public IP `104.21.48.12`). | Passes validation; returns `ValidatedUrlTarget`. |
| **TC-05** | **Negative** | `SSRFValidator` | `test_ssrf_rejects_non_http_scheme` | `file:///etc/passwd`, `ftp://files.com`, `gopher://127.0.0.1` | Raises `UnsupportedSchemeError` (HTTP 400). |
| **TC-06** | **Negative** | `SSRFValidator` | `test_ssrf_rejects_non_web_ports` | `https://example.com:22`, `:5432`, `:6379`, `:8080` | Raises `DisallowedPortError` (HTTP 400). |
| **TC-07** | **Edge-Case** | `SSRFValidator` | `test_ssrf_blocks_aws_gcp_metadata` | `http://169.254.169.254/latest/meta-data/` | Raises `SSRFSecurityViolationError` (HTTP 400). |
| **TC-08** | **Edge-Case** | `SSRFValidator` | `test_ssrf_blocks_ipv4_loopback` | `http://127.0.0.1:80`, `http://127.0.1.1` | Raises `SSRFSecurityViolationError` (HTTP 400). |
| **TC-09** | **Edge-Case** | `SSRFValidator` | `test_ssrf_blocks_ipv6_loopback` | `http://[::1]:80`, `http://[0:0:0:0:0:0:0:1]` | Raises `SSRFSecurityViolationError` (HTTP 400). |
| **TC-10** | **Edge-Case** | `SSRFValidator` | `test_ssrf_blocks_rfc1918_class_a` | `http://10.0.0.1` | Raises `SSRFSecurityViolationError` (HTTP 400). |
| **TC-11** | **Edge-Case** | `SSRFValidator` | `test_ssrf_blocks_rfc1918_class_b` | `http://172.16.0.1`, `http://172.31.255.254` | Raises `SSRFSecurityViolationError` (HTTP 400). |
| **TC-12** | **Edge-Case** | `SSRFValidator` | `test_ssrf_blocks_rfc1918_class_c` | `http://192.168.1.1` | Raises `SSRFSecurityViolationError` (HTTP 400). |
| **TC-13** | **Edge-Case** | `SSRFValidator` | `test_ssrf_blocks_cgnat_rfc6598` | `http://100.64.0.1` | Raises `SSRFSecurityViolationError` (HTTP 400). |
| **TC-14** | **Edge-Case** | `SSRFValidator` | `test_ssrf_blocks_punycode_local` | `http://xn--localhost-1b.com` resolving to `127.0.0.1` | Raises `SSRFSecurityViolationError` (HTTP 400). |
| **TC-15** | **Positive** | `WebsiteExtractor` | `test_extractor_fetches_and_prunes` | MockTransport returns HTML with title, meta, h1-h3, body. | Returns `PrunedDocument` with stripped scripts and styles. |
| **TC-16** | **Negative** | `WebsiteExtractor` | `test_extractor_handles_404_not_found`| MockTransport returns HTTP 404. | Raises `UpstreamWebsiteUnreachableError` (HTTP 502). |
| **TC-17** | **Negative** | `WebsiteExtractor` | `test_extractor_handles_403_cloudflare`| MockTransport returns HTTP 403 Forbidden. | Raises `UpstreamWebsiteUnreachableError` (HTTP 502). |
| **TC-18** | **Negative** | `WebsiteExtractor` | `test_extractor_times_out_after_10s` | MockTransport delays response beyond 10.0s SLA. | Raises `UpstreamWebsiteTimeoutError` (HTTP 504). |
| **TC-19** | **Edge-Case** | `WebsiteExtractor` | `test_extractor_aborts_stream_over_5mb` | MockTransport streams chunks totaling 5,242,881 bytes (>5MB). | Aborts stream immediately; raises `PayloadTooLargeError` (HTTP 413). |
| **TC-20** | **Edge-Case** | `WebsiteExtractor` | `test_extractor_empty_html_body` | MockTransport returns `<html><body></body></html>`. | Checks meta tags; raises `EmptyWebsiteContentError` (HTTP 422). |
| **TC-21** | **Positive** | `DOMPruner` | `test_dom_pruner_caps_character_budget`| Ingests 100,000 characters of raw HTML. | Output text is capped strictly at $\le 32,000$ characters. |
| **TC-22** | **Positive** | `DOMPruner` | `test_dom_pruner_strips_scripts_styles`| HTML containing `<script>alert(1)</script><style>body{}</style>`. | Extracted text contains 0 script or style tags. |
| **TC-23** | **Positive** | `GeminiClient` | `test_gemini_client_synthesizes_5_dimensions`| Valid `PrunedDocument` fed to mocked Gemini 1.5 Flash client. | Returns `LinkAnalysisPayload` with all 5 sections populated. |
| **TC-24** | **Positive** | `GeminiClient` | `test_gemini_zero_hallucinated_volumes`| Inspects generated `TargetSearchQuery` objects. | Zero numeric search volumes, CPC, or difficulty metrics exist. |
| **TC-25** | **Edge-Case** | `GeminiClient` | `test_gemini_prompt_injection_fencing` | Pruned document contains text `Ignore previous instructions`. | XML delimiter ensures prompt executes standard BI analysis. |
| **TC-26** | **Negative** | `CircuitBreaker` | `test_circuit_breaker_trips_on_rate_limit`| Gemini client raises HTTP 429 three consecutive times. | Circuit breaker trips to OPEN; requests route to fallback client. |
| **TC-27** | **Positive** | `OfflineMockClient` | `test_offline_mock_synthesizes_payload`| Unconfigured environment (`GEMINI_API_KEY=""`). | Returns complete synthetic payload with `ai_model="offline-mock"`. |
| **TC-28** | **Positive** | `CacheService` | `test_cache_hit_returns_cached_response`| Ingests identical URL twice within 24-hour TTL window. | Second request returns in <5ms with `cache_hit=True` and $0.00 cost. |
| **TC-29** | **Positive** | `CacheService` | `test_cache_expires_after_ttl` | Freezes clock past 24 hours (`freezegun`). | Cache miss returned; triggers fresh scrape and synthesis. |
| **TC-30** | **Positive** | `AnalysisRouter` | `test_post_link_analysis_success` | Full integration request: `POST /link-analysis {"url": "https://example.com"}`. | HTTP 200 OK matching formal OpenAPI 3.1.0 response contract. |
| **TC-31** | **Negative** | `AnalysisRouter` | `test_post_link_analysis_ssrf_blocked`| Ingress request targeting `http://169.254.169.254`. | HTTP 400 Bad Request with RFC 7807 problem details envelope. |
| **TC-32** | **Positive** | `HealthRouter` | `test_get_health_liveness_probe` | `GET /health` | HTTP 200 OK with `status="healthy"` and uptime metrics. |
| **TC-33** | **Positive** | `HealthRouter` | `test_get_ready_readiness_probe` | `GET /ready` | HTTP 200 OK with `status="ready"` and provider status. |
| **TC-34** | **Positive** | `Middleware` | `test_correlation_id_generated_and_propagated`| Request sent without `X-Correlation-ID` header. | Server generates UUID4, returns in header and log context. |
| **TC-35** | **Positive** | `Middleware` | `test_correlation_id_preserved_from_client` | Request sent with header `X-Correlation-ID: custom-trace-123`. | Response preserves `custom-trace-123` in headers and body telemetry. |

---
*End of Triad Test Catalog. Authored by Piyush Makhija for Dxora Technologies Evaluation.*
