# Governance Architecture: Requirements Traceability Matrix

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  

---

## 1. Traceability Matrix: Dxora Requirements to Code & Tests

| Spec Section | Requirement Description | SRS ID | Design Component | Implementation File | Verification Test |
|:---|:---|:---|:---|:---|:---|
| **Sec 2** | `POST /link-analysis` JSON Endpoint | FR-001 | Ingress Router | `app/routers/analysis.py` | `test_router_analysis.py` |
| **Sec 2 & 9**| Zero-Trust SSRF & IP Blacklist | FR-002, 003 | SSRF Pipeline | `app/utils/ssrf_validator.py` | `test_ssrf_validator.py` |
| **Sec 5** | Async HTTPX Fetch & 5MB Limit | FR-004, 005 | HTTPX Extractor | `app/services/website_extractor.py` | `test_website_extractor.py` |
| **Sec 5 & 6**| DOM Pruner & Token Budget (<8k) | FR-006 | DOM Pruner | `app/services/website_extractor.py` | `test_website_extractor.py` |
| **Sec 9** | XML Prompt Injection Fencing | FR-007 | Prompt Fencer | `app/services/llm_service.py` | `test_llm_service.py` |
| **Sec 3.1** | Section 1: Competitors vs Brand | FR-008 | Gemini Engine | `app/services/llm_service.py` | `test_llm_service.py` |
| **Sec 3.2** | Section 2: 5 Search Queries (0 Vol) | FR-009 | Gemini Engine | `app/services/llm_service.py` | `test_llm_service.py` |
| **Sec 3.3** | Section 3: SEO Keywords & Meta | FR-010 | Gemini Engine | `app/services/llm_service.py` | `test_llm_service.py` |
| **Sec 3.4** | Section 4: Top 5 Services | FR-011 | Gemini Engine | `app/services/llm_service.py` | `test_llm_service.py` |
| **Sec 3.5** | Section 5: Target Audience | FR-012 | Gemini Engine | `app/services/llm_service.py` | `test_llm_service.py` |
| **Sec 2 & 9**| Deterministic Offline Mock Fallback| FR-013 | Mock Adapter | `app/services/llm_service.py` | `test_llm_service.py` |
| **Sec 6.A-C**| LLM Research, Benchmark & Token Economics | NFR-008 | Architecture & Benchmark | `docs/architecture/llm-model-research-benchmark.md` | Benchmark Analysis & Unit Economics |
| **Sec 6.C** | SHA-256 URL Cache (24h TTL) | FR-014 | Cache Engine | `app/services/cache_service.py` | `test_analysis_orchestrator.py`|
| **Sec 8** | RFC 7807 Problem Details Error | FR-015 | Exception Handler | `app/exceptions.py` | `test_router_analysis.py` |
| **Sec 11** | Public Deployment Blueprint | NFR-007 | Render IaC | `render.yaml`, `Dockerfile` | GitHub Actions CI |
