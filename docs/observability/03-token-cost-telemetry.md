# Observability Architecture: Token & Cost Telemetry Specification

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  

---

## 1. Real-Time Token Economics Telemetry

Every successful link analysis response includes an itemized `telemetry` metadata envelope. This provides full visibility into model performance, token expenditure, and dollar cost per request.

### Telemetry Payload Structure
```json
{
  "telemetry": {
    "execution_time_seconds": 1.42,
    "input_tokens_estimated": 7820,
    "output_tokens_estimated": 1410,
    "total_tokens_estimated": 9230,
    "estimated_cost_usd": 0.00101,
    "cost_breakdown": {
      "input_cost_usd": 0.000587,
      "output_cost_usd": 0.000423
    },
    "cache_hit": false,
    "ai_model": "gemini-1.5-flash",
    "correlation_id": "c9a1b2c3-4d5e-6f7a-8b9c-0d1e2f3a4b5c",
    "completed_at": "2026-11-22T07:45:00.542Z"
  }
}
```

---

## 2. Cost Calculation Formula (Gemini 1.5 Flash)

$$\text{Input Cost} = \frac{\text{Input Tokens}}{1,000,000} \times \$0.075$$

$$\text{Output Cost} = \frac{\text{Output Tokens}}{1,000,000} \times \$0.300$$

$$\text{Total Cost USD} = \text{Input Cost} + \text{Output Cost}$$

### Example Benchmark:
- Input Tokens: 8,000 $\to \$0.000600$
- Output Tokens: 1,500 $\to \$0.000450$
- Total per analysis: **$0.001050 USD (~₹0.09 INR)**
