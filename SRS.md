# System Requirements Specification (SRS)
## AI-Powered Website Link Analysis API (`ai-link-analysis`)

> **Standard:** IEEE Std 830-1998 Compliant  
> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Assessment:** Python Practical Test – AI-Powered Website Link Analysis API  
> **Repository:** [https://github.com/piyushmakhija28/ai-link-analysis](https://github.com/piyushmakhija28/ai-link-analysis)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  

---

## 1. Introduction

### 1.1 Purpose
This document provides a formal, comprehensive System Requirements Specification (SRS) for the **AI-Powered Website Link Analysis API** microservice (`ai-link-analysis`). It defines the functional, non-functional, security, architectural, and interface requirements in strict adherence to IEEE Std 830-1998 and Dxora Technologies evaluation guidelines.

### 1.2 Document Conventions
- **MUST / SHALL:** Absolute requirement for system compliance.
- **SHOULD:** Recommended practice; deviation requires architectural justification.
- **MAY:** Optional or configurable capability.

### 1.3 Intended Audience
- Dxora Technologies Technical Evaluation Panel
- Solutions Architects, AppSec Reviewers, and SRE Engineers
- Core API Developers and Quality Assurance Engineers

### 1.4 Project Scope
The `ai-link-analysis` microservice ingests a target public website URL via an asynchronous REST API endpoint, executes zero-trust Server-Side Request Forgery (SSRF) and protocol sanitization, extracts semantic HTML context via non-blocking HTTPX streaming, prunes non-informative DOM nodes to enforce strict token budgets (<8,000 tokens), and orchestrates structured business intelligence generation using **Google Gemini 1.5 Flash**. The service yields a verified 5-section strategic business intelligence report:
1. Competitor vs Brand Details
2. Top 5 Target Search Queries (Google, Facebook, Instagram, LinkedIn, X - Zero Hallucinated Volumes)
3. SEO Improvement Recommendations
4. Top 5 Services / Capabilities
5. Target Audience Identification

---

## 2. Overall Description

### 2.1 Product Perspective
The microservice functions as an independent, stateless REST component in an enterprise business intelligence pipeline. It acts as an intermediary between external client applications and target public websites, safely proxying web extraction and applying generative AI extraction under strict latency, cost, and security bounds.

### 2.2 User Personas
- **P-01: Digital Marketing Strategist:** Needs instant competitor intelligence, search queries, and audience segmentation to structure campaigns.
- **P-02: SEO Analyst:** Requires primary/secondary/long-tail keyword ideas, optimized SEO title/meta recommendations, and content gap strategies.
- **P-03: Business Development Lead:** Seeks rapid comprehension of prospective client capabilities, services, and market advantages.
- **P-04: Automated API Consumer:** Programmatic microservices dispatching high-throughput link evaluation batches.

### 2.3 Operating Environment
- **Runtime:** Python 3.10+ (Tested on Python 3.13)
- **ASGI Framework:** FastAPI 0.115+ running on Uvicorn 0.34+
- **Containerization:** Multi-stage OCI Container (`appuser:10001` non-root)
- **Cloud Host:** Render.com Web Service / Docker Swarm / Kubernetes
- **AI Infrastructure:** Google Gemini 1.5 Flash API with deterministic offline mock fallback

---

## 3. Functional Requirements (FR)

### 3.1 Ingress & Validation (FR-001 .. FR-003)
- **FR-001 (URL Ingress Contract):** The system MUST expose a single REST endpoint `POST http://localhost:4000/link-analysis` accepting a JSON payload `{"url": "https://example.com"}`.
- **FR-002 (URL Syntax & Protocol Sanitization):** The system MUST validate that URLs conform to RFC 3986 syntax and strictly enforce `http` or `https` schemes. Non-HTTP protocols (`file://`, `gopher://`, `ftp://`) MUST be rejected with HTTP 400.
- **FR-003 (Zero-Trust SSRF Defense Pipeline):** The system MUST perform pre-flight socket DNS resolution before dispatching requests. It MUST reject addresses resolving to:
  - Loopback (`127.0.0.0/8`, `::1`)
  - Private subnets (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`)
  - Link-Local / Cloud Metadata (`169.254.0.0/16`, `fe80::/10`, specifically `169.254.169.254`)
  - Carrier-Grade NAT (`100.64.0.0/10`)
  - Multicast/Broadcast ranges
  - Ports other than 80 and 443

### 3.2 Website Extraction & Token Budgeting (FR-004 .. FR-006)
- **FR-004 (Asynchronous HTTP Fetching):** The system MUST use non-blocking HTTPX with connection pooling, following up to 3 redirects, enforcing a 10.0s connection/read timeout SLA.
- **FR-005 (Payload Size Limiter):** The system MUST abort streaming downloads if response body exceeds 5MB to prevent memory exhaustion / decompression bombs (returning HTTP 413).
- **FR-006 (Semantic DOM Pruning & Budgeting):** The system MUST strip `<script>`, `<style>`, `<noscript>`, `<svg>`, `<canvas>`, `<iframe>`, cookie banners, and navigation menus. It MUST isolate high-signal content (`<title>`, `<meta description>`, OpenGraph, JSON-LD, `<h1>`-`<h3>`, `<main>`) and cap output at 32,000 characters (~8,000 tokens).

### 3.3 Prompt Fencing & LLM Intelligence Engine (FR-007 .. FR-012)
- **FR-007 (Prompt Injection Fencing):** Extracted website text MUST be enclosed within `<untrusted_scraped_website_content>` XML tags with explicit system instructions to treat content strictly as inert data.
- **FR-008 (Section 1 - Competitor vs Brand Details):** The LLM response MUST generate:
  - Brand name and business overview
  - Main business category and primary products/services
  - Minimum 3 identified direct or industry competitors
  - Competitor strengths and gaps/weaknesses
  - Brand's competitive advantages and market opportunities
- **FR-009 (Section 2 - Top 5 Target Search Queries):** The LLM response MUST generate exactly 5 search queries across Google, Facebook, Instagram, LinkedIn, and X, specifying search intent, business relevance, and target audience. **Constraint: The system MUST NOT fabricate or hallucinate numerical monthly search volumes.**
- **FR-010 (Section 3 - SEO Improvement):** The LLM response MUST generate:
  - Primary, secondary, and long-tail keyword recommendations
  - Recommended SEO title structure
  - Human-friendly meta description (150-160 characters)
  - Additional supporting content recommendations (landing pages, FAQs, topic clusters)
- **FR-011 (Section 4 - Top 5 Services):** The LLM response MUST extract the top 5 services, identifying service name, description, target industry, target customer, and main benefit. Inferred services MUST be explicitly labeled.
- **FR-012 (Section 5 - Target Audience Identification):** The LLM response MUST segment primary audience, secondary audience, industry verticals, customer profiles, geographic focus, and core customer pain points.

### 3.4 Resilience & Offline Fallback (FR-013 .. FR-015)
- **FR-013 (Deterministic Offline Mock Fallback):** If `GEMINI_API_KEY` is absent or upstream rate-limited (HTTP 429), the system MUST gracefully fall back to a deterministic offline rule-based extraction engine, returning complete diagnostic intelligence without failing the request.
- **FR-014 (URL-Hash Response Caching):** The system MUST compute a SHA-256 hash of the normalized URL and support caching responses for 24 hours (`CACHE_TTL=86400`), serving cache hits in <5ms.
- **FR-015 (RFC 7807 Error Handling):** All HTTP client and server errors MUST return structured `application/problem+json` envelopes containing `type`, `title`, `status`, `detail`, `instance`, and `correlation_id`.

---

## 4. Non-Functional Requirements (NFR)

- **NFR-001 (Performance & Latency):** Live website analysis end-to-end latency MUST not exceed 5.0 seconds for normal web pages under Gemini 1.5 Flash; cached requests MUST return in under 20ms.
- **NFR-002 (Cost Efficiency):** API token usage MUST not exceed $0.002 per live request (~₹0.15 INR) via single-pass multi-section prompt consolidation.
- **NFR-003 (Reliability & Availability):** The microservice MUST maintain 99.9% uptime with independent bulkhead fault isolation preventing unhandled task crashes.
- **NFR-004 (Code & Docstring Standards):** Strict adherence to Claude Global Rule 12 (Level 2.2 Docstrings-Only). Zero `#` inline narration comments. 100% PEP 257 docstring coverage across public modules, classes, and methods.
- **NFR-005 (Type Safety):** 100% type coverage passing `mypy --strict` with zero type errors.
- **NFR-006 (Test Coverage):** Pytest triad testing suite (positive, negative, edge-case) MUST meet or exceed **90% statement coverage**.
- **NFR-007 (Security & Privacy):** Zero hardcoded credentials; non-root Docker container (`appuser:10001`); Bandit SAST security scans returning 0 high/medium issues.
- **NFR-008 (Observability):** Centralized structured JSON logging with correlation ID tracing (`X-Correlation-ID`) across all requests, logs, and responses.
