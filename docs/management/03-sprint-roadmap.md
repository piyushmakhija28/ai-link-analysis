# Governance Architecture: Sprint Roadmap & Milestone Delivery

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  

---

## 1. Sprint Roadmap across 6 Milestones

```mermaid
gantt
    title ai-link-analysis 6-Sprint Delivery Roadmap
    dateFormat YYYY-MM-DD
    section Sprint 1
    Docs & Architecture Baseline (Zero Code) :done, s1, 2026-11-22, 1d
    section Sprint 2
    Ingress, Schemas & Zero-Trust SSRF :active, s2, 2026-11-23, 1d
    section Sprint 3
    Async Extractor & DOM Pruner :s3, 2026-11-24, 1d
    section Sprint 4
    Gemini 1.5 Flash & BI Extraction :s4, 2026-11-25, 1d
    section Sprint 5
    Orchestrator, Routers & Triad Tests :s5, 2026-11-26, 1d
    section Sprint 6
    Docker OCI, GitHub CI & Render Deploy :s6, 2026-11-27, 1d
```

---

## 2. Milestone Deliverables Summary

- **Milestone 1 (Sprint 1):** Architecture Baseline, IEEE 830 SRS, 8 ADRs, 13 Mermaid UML diagrams, 8 Draw.io diagrams, OpenAPI 3.1.0 contract.
- **Milestone 2 (Sprint 2):** Config, constants, domain schemas, RFC 7807 problem details, SSRF validator with triad unit tests.
- **Milestone 3 (Sprint 3):** Async HTTPX streaming client, BeautifulSoup4 semantic pruner, token budgeting, with mock transport triad tests.
- **Milestone 4 (Sprint 4):** Google GenAI SDK integration, 5-in-1 consolidated prompt, offline mock fallback, and cost telemetry with triad tests.
- **Milestone 5 (Sprint 5):** Analysis orchestrator, routers (`/link-analysis`, `/health`, `/ready`), correlation ID middleware, full triad tests achieving $\ge 90\%$ coverage.
- **Milestone 6 (Sprint 6):** Multi-stage non-root Dockerfile, `docker-compose.yml`, GitHub Actions CI workflow, Render.com blueprint, live demo verification.
