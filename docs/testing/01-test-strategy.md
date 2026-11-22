# Testing Architecture: 11-Layer Test Strategy

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  

---

## 1. Testing Philosophy & Test Pyramid

The microservice implements a strict **11-Layer Testing Pyramid** following enterprise standards and `41-testing-standards-python-fastapi.md`. Testing is hermetic, deterministic, and executes in <5 seconds without external network dependencies.

```mermaid
graph TD
    subgraph Pyramid["11-Layer Testing Pyramid"]
        E2E["Layer 11: End-to-End Integration Tests (test_e2e_flow.py)"]
        RouterAnalysis["Layer 10: Router Endpoint Tests (POST /link-analysis)"]
        RouterHealth["Layer 9: Health & Readiness Probe Tests (GET /health, /ready)"]
        Orchestrator["Layer 8: Analysis Orchestrator Resilience & Bulkhead Tests"]
        LLMRepair["Layer 7: LLM Schema Validation & JSON Repair Loop Tests"]
        LLMClient["Layer 6: Gemini Client & Offline Mock Fallback Tests"]
        DOMPruner["Layer 5: DOM Pruner & Token Budgeting (<8k) Tests"]
        Extractor["Layer 4: Async HTTPX Extractor & 5MB Payload Limit Tests"]
        SSRF["Layer 3: Zero-Trust SSRF & DNS Pre-Flight Tests"]
        Schemas["Layer 2: Pydantic v2 Schema & RFC 7807 Error Model Tests"]
        Config["Layer 1: Settings, Constants & Bootstrap Tests"]
    end
```

---

## 2. Triad Testing Methodology

Every critical layer requires three distinct test dimensions:
1. **Positive Tests (Happy Path):** Valid inputs, successful extraction, schema compliance, HTTP 200 responses.
2. **Negative Tests (Error Path):** Invalid URLs, unreachable hosts, HTTP 401/403/404/500 upstream errors, AI rate-limiting (HTTP 429), malformed JSON.
3. **Edge-Case Tests (Adversarial & Boundary Path):** SSRF attacks (`169.254.169.254`, `127.0.0.1`, IPv6 loopback, internal ports), prompt injection attempts, massive 5MB+ payloads, empty HTML without body or title.

---

## 3. Coverage Gate & CI/CD Enforcement
- **Statement Coverage Threshold:** Minimum **90%** enforced by pytest-cov (`--cov-fail-under=90`).
- **Speed SLA:** Entire test suite runs in under 5.0 seconds via in-memory HTTPX `MockTransport` fixtures.
