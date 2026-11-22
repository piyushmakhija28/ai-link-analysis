# Governance Architecture: Service Directed Acyclic Graph (DAG)

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  

---

## 1. Service Directed Acyclic Graph (DAG)

The internal execution graph is strictly acyclic with defined error boundaries:

```mermaid
graph TD
    A["Node 1: Request Ingress (POST /link-analysis)"] --> B["Node 2: Correlation ID Context Injection"]
    B --> C{"Node 3: SHA-256 Cache Check"}
    C -->|"Cache Hit (<5ms)"| R["Node 11: Return Cached JSON Response"]
    C -->|"Cache Miss"| D["Node 4: Pydantic Syntax & Scheme Check"]
    D --> E["Node 5: Pre-Flight DNS & SSRF Subnet Filter"]
    E --> F["Node 6: Async HTTPX Stream Fetch (<5MB cap, 10s SLA)"]
    F --> G["Node 7: DOM Pruning & Character Budgeting (<32k chars)"]
    G --> H["Node 8: Prompt Fencing (<untrusted_scraped_website_content>)"]
    H --> I["Node 9: Gemini 1.5 Flash Invocation (Single-Pass 5-in-1)"]
    I --> J["Node 10: Pydantic v2 Schema Validation & JSON Repair"]
    J --> K["Node 11: Cache Store (SHA-256 key, 24h TTL)"]
    K --> L["Node 12: Telemetry Enrichment (Latency, Tokens, Cost USD)"]
    L --> M["Node 13: HTTP 200 OK Delivery"]

    subgraph ErrorBoundaries["Error Boundaries (RFC 7807 Problem Details)"]
        D --x|Scheme Error| EB1["400 Invalid Scheme"]
        E --x|SSRF Target| EB2["400 SSRF Violation"]
        F --x|HTTP 404/500| EB3["502 Website Unreachable"]
        F --x|Payload >5MB| EB4["413 Payload Too Large"]
        F --x|10s Timeout| EB5["504 Gateway Timeout"]
        I --x|Rate Limited| EB6["Graceful Fallback to Offline Mock"]
    end
```
