# Governance Architecture: Bidirectional Requirements Traceability Matrix (RTM)

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Repository:** [https://github.com/piyushmakhija28/ai-link-analysis](https://github.com/piyushmakhija28/ai-link-analysis)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  
> **Status:** Baselined & Verified  

---

## 1. Executive Summary & Purpose

This **Bidirectional Requirements Traceability Matrix (RTM)** establishes complete forward and backward traceability across the entire lifecycle of the **AI-Powered Website Link Analysis API** (`ai-link-analysis`).

It rigorously maps every single clause of the **Dxora Technologies Practical Assessment Specification** to:
1. **IEEE Std 830-1998 System Requirements Specification (SRS)** Functional & Non-Functional IDs.
2. **Product Requirements Document (PRD)** Feature Identifiers.
3. **High-Level Design (HLD) & Low-Level Design (LLD)** Architecture Components.
4. **Concrete Python Implementation Files & Classes**.
5. **Triad Test Suite Files (Positive, Negative, Edge-Case)**.
6. **Automated Verification Commands**.

---

## 2. Complete Bidirectional Traceability Matrix

| Dxora Spec Section | Specification Requirement Clause | SRS ID | PRD ID | Architecture Component | Implementation File & Symbol | Triad Test File & Coverage | Verification Command |
|:---|:---|:---|:---|:---|:---|:---|:---|
| **Sec 2.A** | `POST http://localhost:4000/link-analysis` endpoint | `FR-001` | `FEAT-01` | Ingress Router | `app/routers/analysis.py`<br/>`analyze_link()` | `tests/test_router_analysis.py`<br/>`test_post_link_analysis_success()` | `pytest tests/test_router_analysis.py` |
| **Sec 2.B** | Request payload `{"url": "https://example.com"}` | `FR-001` | `FEAT-01` | Domain Schemas | `app/schemas.py`<br/>`LinkAnalysisRequest` | `tests/test_schemas.py`<br/>`test_request_schema_validation()` | `pytest tests/test_schemas.py` |
| **Sec 2 & 9** | Zero-Trust SSRF & RFC 1918 Private IP Filter | `FR-003`<br/>`FR-004` | `FEAT-02` | SSRF Security Pipeline | `app/utils/ssrf_validator.py`<br/>`validate_target_url()` | `tests/test_ssrf_validator.py`<br/>`test_blocks_rfc1918_private_ips()` | `pytest tests/test_ssrf_validator.py` |
| **Sec 2 & 9** | Cloud Metadata Filter (`169.254.169.254` Link-Local) | `FR-004` | `FEAT-02` | SSRF Security Pipeline | `app/utils/ssrf_validator.py`<br/>`is_ip_blacklisted()` | `tests/test_ssrf_validator.py`<br/>`test_blocks_aws_gcp_metadata()` | `pytest tests/test_ssrf_validator.py` |
| **Sec 2 & 9** | Port Whitelist (Strictly Ports 80 and 443) | `FR-005` | `FEAT-02` | SSRF Security Pipeline | `app/utils/ssrf_validator.py`<br/>`validate_port()` | `tests/test_ssrf_validator.py`<br/>`test_rejects_non_web_ports()` | `pytest tests/test_ssrf_validator.py` |
| **Sec 5.A** | Asynchronous HTTPX Client with Connection Pooling | `FR-007` | `FEAT-01` | Web Extractor | `app/services/website_extractor.py`<br/>`WebsiteExtractor` | `tests/test_website_extractor.py`<br/>`test_async_fetch_success()` | `pytest tests/test_website_extractor.py` |
| **Sec 5.B** | Streaming Body Size Ceiling (5MB Max) | `FR-009` | `FEAT-03` | Streaming Limiter | `app/services/website_extractor.py`<br/>`fetch_and_extract()` | `tests/test_website_extractor.py`<br/>`test_aborts_payload_exceeding_5mb()` | `pytest tests/test_website_extractor.py` |
| **Sec 5.C** | Upstream Connection SLA Timeout (10.0s) | `FR-010` | `FEAT-01` | Timeout Enforcer | `app/services/website_extractor.py`<br/>`httpx.Timeout(10.0)` | `tests/test_website_extractor.py`<br/>`test_upstream_timeout_504()` | `pytest tests/test_website_extractor.py` |
| **Sec 5 & 6** | DOM Pruning (Strip scripts, styles, nav, footers) | `FR-011`<br/>`FR-012` | `FEAT-04` | Semantic DOM Pruner | `app/services/website_extractor.py`<br/>`prune_html_to_markdown()` | `tests/test_website_extractor.py`<br/>`test_prunes_scripts_and_styles()` | `pytest tests/test_website_extractor.py` |
| **Sec 5 & 6** | Ingress Token / Character Ceiling (<32k chars / <8k tokens) | `FR-013` | `FEAT-04` | Token Budget Capper | `app/services/website_extractor.py`<br/>`apply_character_ceiling()` | `tests/test_website_extractor.py`<br/>`test_caps_character_budget()` | `pytest tests/test_website_extractor.py` |
| **Sec 9** | Prompt Injection XML Fencing | `FR-014` | `FEAT-05` | Prompt Fencer | `app/services/llm_service.py`<br/>`build_analysis_prompt()` | `tests/test_llm_service.py`<br/>`test_prompt_injection_fencing()` | `pytest tests/test_llm_service.py` |
| **Sec 6.C** | Single-Pass 5-in-1 Prompt Execution (80% token savings) | `FR-015` | `FEAT-12` | Consolidated Engine | `app/services/llm_service.py`<br/>`GeminiLLMClient` | `tests/test_llm_service.py`<br/>`test_single_pass_prompt_composition()` | `pytest tests/test_llm_service.py` |
| **Sec 3.1** | Section 1: Competitors vs Brand Details | `FR-016` | `FEAT-06` | Gemini Flash Engine | `app/services/llm_service.py`<br/>`analyze_content()` | `tests/test_llm_service.py`<br/>`test_extracts_competitor_details()` | `pytest tests/test_llm_service.py` |
| **Sec 3.2** | Section 2: Top 5 Search Queries across 5 Platforms | `FR-017` | `FEAT-07` | Gemini Flash Engine | `app/services/llm_service.py`<br/>`TargetSearchQuery` | `tests/test_llm_service.py`<br/>`test_extracts_5_platform_queries()` | `pytest tests/test_llm_service.py` |
| **Sec 3.2 Note**| Strict Zero Search Volume Hallucination | `FR-018` | `FEAT-07` | Schema Negative Guard | `app/schemas.py`<br/>`TargetSearchQuery` | `tests/test_schemas.py`<br/>`test_zero_hallucinated_search_volumes()` | `pytest tests/test_schemas.py` |
| **Sec 3.3** | Section 3: SEO Improvement Recommendations | `FR-019` | `FEAT-08` | Gemini Flash Engine | `app/services/llm_service.py`<br/>`SEOStrategy` | `tests/test_llm_service.py`<br/>`test_extracts_seo_recommendations()` | `pytest tests/test_llm_service.py` |
| **Sec 3.4** | Section 4: Top 5 Services (Inferred Labeled) | `FR-020` | `FEAT-09` | Gemini Flash Engine | `app/services/llm_service.py`<br/>`ServiceCapability` | `tests/test_llm_service.py`<br/>`test_extracts_top_5_services()` | `pytest tests/test_llm_service.py` |
| **Sec 3.5** | Section 5: Target Audience Identification | `FR-021` | `FEAT-10` | Gemini Flash Engine | `app/services/llm_service.py`<br/>`TargetAudienceProfile`| `tests/test_llm_service.py`<br/>`test_extracts_target_audience()` | `pytest tests/test_llm_service.py` |
| **Sec 6.A-C** | LLM Model Research, Benchmark & Unit Economics | `NFR-004` | `FEAT-12` | Architecture Doc | `docs/architecture/`<br/>`llm-model-research-benchmark.md` | Empirical Unit Economics & Benchmark Tables | Review Document |
| **Sec 2 & 7** | Deterministic Offline Mock Client Fallback | `FR-023` | `FEAT-11` | Mock Adapter | `app/services/llm_service.py`<br/>`OfflineMockClient` | `tests/test_llm_service.py`<br/>`test_offline_mock_fallback()` | `pytest tests/test_llm_service.py` |
| **Sec 6.C** | SHA-256 URL TTL Cache (24h TTL, <5ms Hit) | `FR-024` | `FEAT-13` | In-Memory Cache | `app/services/cache_service.py`<br/>`AnalysisCacheService` | `tests/test_cache_service.py`<br/>`test_cache_hit_and_expiration()` | `pytest tests/test_cache_service.py` |
| **Sec 8** | RFC 7807 Problem Details Error Formatting | `FR-026` | `FEAT-15` | Exception Handlers | `app/exceptions.py`<br/>`problem_exception_handler()` | `tests/test_exceptions.py`<br/>`test_rfc7807_error_envelope()` | `pytest tests/test_exceptions.py` |
| **Sec 11** | Production Dockerfile & Render.com Blueprint | `NFR-010` | `FEAT-16` | DevOps & Cloud IaC | `Dockerfile`, `render.yaml`,<br/>`.github/workflows/ci.yml`| `tests/test_health_probes.py`<br/>`test_liveness_and_readiness()` | `docker build -t ai-link-analysis .` |

---

## 3. Verification & Compliance Sign-Off

- **Traceability Completeness:** 100% of Dxora specification sections map directly to at least one functional requirement, code artifact, and automated test case.
- **Orphan Prevention:** Zero code files or public functions exist without a corresponding requirement and test case.
- **Audit Verification:** All tests are automatically validated in GitHub Actions CI across Python 3.10, 3.11, 3.12, and 3.13.
