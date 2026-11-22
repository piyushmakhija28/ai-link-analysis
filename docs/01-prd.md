# Product Requirements Document (PRD)
## AI-Powered Website Link Analysis API (`ai-link-analysis`)

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Product Stage:** Production-Ready Enterprise Microservice  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  

---

## 1. Problem Definition & Market Opportunity

### 1.1 The Core Problem
In competitive business intelligence, digital marketing, and sales prospecting, understanding an unfamiliar website manually takes anywhere from 30 minutes to 2 hours per company. Analysts must manually browse multiple pages, discern the company's core positioning, identify who they compete against, craft SEO keyword recommendations, infer the target audience, and determine which search queries drive traffic across various platforms.

Doing this at scale across hundreds of client sites or competitor portfolios is cost-prohibitive and slow.

### 1.2 The Solution
The **AI-Powered Website Link Analysis API** automates this entire discovery cycle in under **2 seconds**. By providing a target website URL (e.g. `https://example.com`), client applications instantly receive a deeply structured, 5-dimensional strategic business intelligence dossier powered by Google Gemini 1.5 Flash.

---

## 2. Target Personas & User Journeys

### 2.1 Target Personas
```
┌─────────────────────────────────┐       ┌─────────────────────────────────┐
│ Persona A: Digital Strategist   │       │ Persona B: SEO Consultant       │
├─────────────────────────────────┤       ├─────────────────────────────────┤
│ Goal: Quick competitor research │       │ Goal: Keyword & meta gap audit  │
│ Need: 3+ competitor comparison, │       │ Need: Primary/long-tail keywords│
│ platform-specific search queries│       │ title & meta description fixes  │
└─────────────────────────────────┘       └─────────────────────────────────┘
┌─────────────────────────────────┐       ┌─────────────────────────────────┐
│ Persona C: B2B Sales Lead       │       │ Persona D: SaaS Integration App │
├─────────────────────────────────┤       ├─────────────────────────────────┤
│ Goal: Prospective client review │       │ Goal: Programmatic batch link   │
│ Need: Inferred customer pain    │       │ evaluation with low latency and │
│ points, services, and benefits  │       │ cost predictable at scale       │
└─────────────────────────────────┘       └─────────────────────────────────┘
```

### 2.2 Primary User Journey
1. **User Action:** Client sends a POST request with `{"url": "https://example.com"}` to `/link-analysis`.
2. **System Ingress:** Security pipeline verifies URL protocol and validates IP address safety against SSRF and cloud metadata theft.
3. **Extraction & Pruning:** HTTPX streams HTML, prunes noise (scripts, styles, navbars), and compresses semantic business content to under 8,000 tokens.
4. **LLM Synthesis:** Gemini 1.5 Flash analyzes content using a single consolidated prompt with strict schema enforcement.
5. **Client Value Delivery:** Within ~1.5s, client receives a 200 OK JSON report covering Brand/Competitors, 5-Platform Search Queries, SEO Strategy, Top 5 Services, and Audience Segmentation.

---

## 3. Product Scope & Functional Feature Matrix

| Feature ID | Feature Name | Description | Priority |
|:---|:---|:---|:---:|
| **FEAT-01** | Single Ingress Endpoint | REST API `POST http://localhost:4000/link-analysis` accepting `{"url": "..."}`. | P0 (Mandatory) |
| **FEAT-02** | Zero-Trust SSRF Shield | Rejects private subnets, cloud metadata (`169.254.169.254`), loopbacks, and non-80/443 ports. | P0 (Mandatory) |
| **FEAT-03** | Semantic DOM Pruner | Strips boilerplate, navigations, footers; budget capped at <8k tokens (~32k chars). | P0 (Mandatory) |
| **FEAT-04** | Prompt Injection Fencing | Encapsulates scraped text inside XML fences `<untrusted_scraped_website_content>`. | P0 (Mandatory) |
| **FEAT-05** | Section 1: Competitor Intelligence | Brand overview, category, $\ge 3$ competitors, strengths, weaknesses, advantages, opportunities. | P0 (Mandatory) |
| **FEAT-06** | Section 2: 5-Platform Search Queries | 5 queries across Google, Facebook, Instagram, LinkedIn, X (Zero fabricated search volumes). | P0 (Mandatory) |
| **FEAT-07** | Section 3: SEO Improvement Engine | Primary/secondary/long-tail keywords, SEO title, meta description, and supporting landing page ideas. | P0 (Mandatory) |
| **FEAT-08** | Section 4: Top 5 Services | Service name, description, target industry, target customer, main benefit. Inferred items labeled. | P0 (Mandatory) |
| **FEAT-09** | Section 5: Target Audience | Primary/secondary audience, industry segments, customer profiles, geographic focus, pain points. | P0 (Mandatory) |
| **FEAT-10** | Deterministic Offline Mock | Graceful diagnostic fallback when `GEMINI_API_KEY` is not configured or rate-limited. | P0 (Mandatory) |
| **FEAT-11** | SHA-256 URL Cache | Caches analysis results for 24 hours (`CACHE_TTL=86400`), serving cache hits in <5ms. | P1 (High) |
| **FEAT-12** | Telemetry & Cost Transparency | Enriches responses with execution time, token estimates, and calculated USD cost. | P1 (High) |
| **FEAT-13** | SRE Health Probes | Exposes `/health` (liveness) and `/ready` (readiness) for container orchestrators. | P1 (High) |

---

## 4. Business & Technical Constraints

1. **Zero Search Volume Hallucination:** Under Section 3 Note and Section 9 of the Dxora specification, numerical monthly search volumes must NEVER be fabricated or hallucinated. Search queries must focus on qualitative search intent, platform suitability, and audience relevance.
2. **Cost Optimization:** Must utilize **Google Gemini 1.5 Flash** ($0.075/1M input, $0.30/1M output), executing a single multi-section query to ensure per-analysis cost remains around **$0.00105 (under 10 paise INR)**.
3. **Execution SLA:** Upstream website fetching must time out strictly after 10.0 seconds; LLM processing must complete within 5.0 seconds.

---

## 5. Success Metrics (KPIs)

- **Accuracy & Grounding:** 100% of insights grounded in actual website content or clearly labeled as inferred.
- **Latency SLA:** Live website analysis median latency $\le 2.0$ seconds; 95th percentile $\le 3.5$ seconds.
- **Cost Efficiency:** Average API cost $\le \$0.0015$ per live request.
- **Security:** 0 SSRF bypasses, 0 unhandled 500 server crashes.
- **Code Quality:** $\ge 90\%$ statement test coverage, 0 mypy strict errors, 0 ruff lint errors.
