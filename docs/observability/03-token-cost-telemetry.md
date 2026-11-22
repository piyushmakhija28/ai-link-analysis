# Observability Architecture: Token Economics & Cost Telemetry Specification

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Repository:** [https://github.com/piyushmakhija28/ai-link-analysis](https://github.com/piyushmakhija28/ai-link-analysis)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  
> **Status:** Baselined & Enforced  

---

## 1. Executive Summary & Objective

In enterprise AI engineering, transparency into generative model token consumption and operational cost is critical. Client applications and evaluators need clear auditability into:
- Exactly how many input and output tokens were consumed for each website link analyzed.
- The precise cost incurred in both US Dollars (USD) and Indian Rupees (INR).
- The execution latency breakdown between network scraping and AI synthesis.
- Whether the result was served from the sub-millisecond SHA-256 URL cache ($0.00 cost) or generated live.

Under Section 6 of the Dxora Technologies specification, the API enriches every response with a formal `telemetry` envelope:

```json
{
  "telemetry": {
    "execution_time_seconds": 1.45,
    "input_tokens_estimated": 7820,
    "output_tokens_estimated": 1410,
    "total_tokens_estimated": 9230,
    "estimated_cost_usd": 0.00101,
    "estimated_cost_inr": 0.08686,
    "cost_breakdown": {
      "input_cost_usd": 0.000587,
      "output_cost_usd": 0.000423
    },
    "cache_hit": false,
    "ai_provider": "google-gemini",
    "ai_model": "gemini-1.5-flash",
    "correlation_id": "c9a1b2c3-4d5e-6f7a-8b9c-0d1e2f3a4b5c",
    "completed_at": "2026-11-22T07:45:00.542100Z"
  }
}
```

---

## 2. Real-Time Cost Calculation Algorithms

### 2.1 Gemini 1.5 Flash Mathematical Formulation
Current pricing tier ($\le 128\text{k tokens}$ context window):
- **Input Tokens Rate ($R_{\text{in}}$):** $\$0.075 \text{ per } 1,000,000 \text{ tokens} = \$0.000000075 / \text{token}$
- **Output Tokens Rate ($R_{\text{out}}$):** $\$0.300 \text{ per } 1,000,000 \text{ tokens} = \$0.000000300 / \text{token}$
- **Exchange Baseline ($E$):** $\$1.00 \text{ USD} = ₹86.00 \text{ INR}$

$$\text{Input Cost (USD)} = T_{\text{in}} \times R_{\text{in}} = 7,820 \times 0.000000075 = \$0.0005865$$

$$\text{Output Cost (USD)} = T_{\text{out}} \times R_{\text{out}} = 1,410 \times 0.000000300 = \$0.0004230$$

$$\text{Total Cost (USD)} = \text{Input Cost} + \text{Output Cost} = \$0.0010095 \approx \$0.00101$$

$$\text{Total Cost (INR)} = \text{Total Cost (USD)} \times 86.00 = ₹0.08686 \text{ INR (~8.7 Paise)}$$

### 2.2 Cache Hit Cost Calculation
When `cache_hit == True`:
$$\text{Total Cost (USD)} = \$0.0000000$$
$$\text{Total Cost (INR)} = ₹0.0000000$$
$$\text{Execution Latency} \le 0.005\text{ seconds (5ms)}$$

---

## 3. Prometheus Metric Declarations

The service exposes Prometheus-compatible metric instrumentation:
- `llm_requests_total{provider, model, status}`: Counter tracking total LLM requests.
- `llm_cost_usd_total{provider, model}`: Counter accumulating total financial expenditure.
- `llm_tokens_total{provider, model, type="input|output"}`: Counter tracking token consumption.
- `website_scrape_latency_seconds`: Histogram measuring upstream website fetch latency.
- `llm_synthesis_latency_seconds`: Histogram measuring generative inference latency.
- `cache_hits_total` / `cache_misses_total`: Counters measuring cache efficiency.

---
*End of Token Economics & Cost Telemetry Specification. Authored by Piyush Makhija for Dxora Technologies Evaluation.*
