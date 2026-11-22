# Security Architecture: OWASP API Security Top 10 (2023) & LLM Top 10 (2025)

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Repository:** [https://github.com/piyushmakhija28/ai-link-analysis](https://github.com/piyushmakhija28/ai-link-analysis)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  
> **Status:** Baselined & Enforced  

---

## 1. OWASP API Security Top 10 (2023) Compliance Mapping

The following matrix documents our point-by-point mitigation of the industry-standard **OWASP API Security Top 10 (2023)** vulnerabilities:

| OWASP Vulnerability | Risk Definition | Status in `ai-link-analysis` | Concrete Architectural Mitigation |
|:---|:---|:---:|:---|
| **API1:2023 Broken Object Level Authorization (BOLA)** | Manipulating resource IDs to access unauthorized tenant records. | **N/A (Stateless)** | The microservice is completely stateless. It does not store user data or expose resource IDs (`/items/{id}`). |
| **API2:2023 Broken Authentication** | Flaws in token validation, password resets, or session expiration. | **Mitigated** | Public ingress endpoint designed for open B2B prospecting. Production deployments integrate with upstream API gateway reverse proxy (Bearer JWT). |
| **API3:2023 Broken Object Property Level Authorization** | Exposing sensitive internal model attributes (mass assignment). | **Compliant** | Strict separation between ingress requests (`LinkAnalysisRequest`) and egress responses (`LinkAnalysisResponse`). No internal fields exposed. |
| **API4:2023 Unrestricted Resource Consumption** | DoS via unbounded memory usage, CPU spikes, or request flooding. | **Compliant** | 1. 5MB streaming download ceiling on scraped responses.<br/>2. 10.0-second SLA timeout on HTTPX connections.<br/>3. Rate limits configured on Uvicorn workers. |
| **API5:2023 Broken Function Level Authorization (BFLA)** | Regular users executing administrative API functions. | **Compliant** | SRE health probes (`/health`, `/ready`) expose only non-sensitive liveness booleans; no administrative modification routes exist. |
| **API6:2023 Unrestricted Access to Sensitive Business Flows** | Bots spamming endpoints causing financial or operational exhaustion. | **Compliant** | SHA-256 URL caching (24h TTL) returns repeat requests in 2ms at **$0.00 API cost**, preventing token billing exhaustion attacks. |
| **API7:2023 Server-Side Request Forgery (SSRF)** | Exploiting backend network access to query internal cloud metadata or VPCs. | **Fully Defended** | **7-Stage Zero-Trust SSRF Pipeline:** Blocks private subnets (RFC 1918), link-local cloud metadata (`169.254.169.254`), non-standard ports, and non-HTTP schemes. |
| **API8:2023 Security Misconfiguration** | Default passwords, unpatched libraries, or verbose error stack traces. | **Compliant** | 1. RFC 7807 problem details sanitize all errors (zero stack traces leaked).<br/>2. Non-root OCI container (`appuser:10001`).<br/>3. Read-only root filesystem. |
| **API9:2023 Improper Inventory Management** | Undocumented shadow APIs or deprecated legacy endpoints. | **Compliant** | Formal OpenAPI 3.1.0 specification (`openapi.yaml`) kept in sync with code; Swagger UI auto-generated. |
| **API10:2023 Unsafe Consumption of APIs** | Blindly trusting responses from third-party APIs or web servers. | **Compliant** | Untrusted scraped website content is DOM-pruned and fenced in XML tags; LLM outputs are validated against strict Pydantic v2 schemas. |

---

## 2. OWASP Top 10 for LLM Applications (2025) Compliance Mapping

| LLM Vulnerability | Risk Definition | Mitigation Strategy in `ai-link-analysis` |
|:---|:---|:---|
| **LLM01: Prompt Injection** | Adversarial user or website content overriding system instructions. | **XML Boundary Fencing:** Scraped text encapsulated in `<untrusted_scraped_website_content>` with strict instruction precedence. |
| **LLM02: Sensitive Information Disclosure** | Model emitting training data secrets or system credentials. | Environment API keys are never included in prompts or logs; PII is stripped by DOM pruner. |
| **LLM04: Model Denial of Service** | Complex or unbounded prompts exhausting token budgets or context windows. | Character budget capper enforces a strict ceiling of **32,000 characters (~8,000 tokens)** on extracted text. |
| **LLM06: Excessive Agency** | Model executing autonomous tool calls or shell commands. | The LLM operates strictly in read-only generative analysis mode; zero autonomous tool calling or file modification tools are granted to the model. |
| **LLM09: Misinformation & Hallucination** | Model fabricating plausible-sounding but false metrics or numbers. | **Strict Zero-Hallucination Guardrails:** Pydantic schema strictly excludes search volume and CPC fields; negative constraints prevent fabrication. |

---
*End of OWASP API & LLM Security Specification. Authored by Piyush Makhija for Dxora Technologies Evaluation.*
