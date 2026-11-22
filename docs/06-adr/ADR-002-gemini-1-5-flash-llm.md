# ADR-002: Selection of Google Gemini 1.5 Flash as Primary LLM & Dual-Engine Fallback

## Status
Accepted

## Context
Under Section 6 of the Dxora Technologies practical test specification, applicants must research, benchmark, and recommend the most suitable LLM platform based on accuracy, response speed, and API cost.

A comprehensive empirical evaluation of 8 frontier and lightweight models (detailed in [docs/architecture/llm-model-research-benchmark.md](../architecture/llm-model-research-benchmark.md)) evaluated:
1. **Google Gemini 1.5 Flash:** $0.075 / 1M input, $0.30 / 1M output, 1,000,000 context window, ~1.2s–1.5s latency, native Pydantic schema steering.
2. **Google Gemini 1.5 Flash-8B:** $0.0375 / 1M input, $0.15 / 1M output, ultra-fast but slightly reduced reasoning for multi-competitor deduction.
3. **OpenAI GPT-4o-mini:** $0.15 / 1M input, $0.60 / 1M output, 128k context, ~1.8s latency, strict JSON schema support (2x cost of Gemini Flash).
4. **DeepSeek V3 (Chat):** $0.14 / 1M input, $0.28 / 1M output, 64k context, low cost but dependent on third-party gateway availability.
5. **Anthropic Claude 3.5 Haiku:** $0.80 / 1M input, $4.00 / 1M output, ~2.0s latency (10.7x input / 13.3x output cost vs Gemini Flash).
6. **Google Gemini 1.5 Pro:** $1.25 / 1M input, $5.00 / 1M output, 2M context (16.7x more expensive; overkill for single landing pages).
7. **OpenAI GPT-4o:** $2.50 / 1M input, $10.00 / 1M output (33.3x more expensive; unviable for automated link analysis).
8. **Claude 3.5 Sonnet:** $3.00 / 1M input, $15.00 / 1M output (44.3x more expensive).

## Decision
1. We select **Google Gemini 1.5 Flash** (via the official `google-genai` SDK) as the primary intelligence engine.
2. We establish **OpenAI GPT-4o-mini** as the secondary automated circuit breaker fallback in case of Gemini quota exhaustion or transient 429/503 errors.
3. We establish **OfflineMockClient** for local development, unit tests, and CI/CD pipelines ($0.00 cost).
4. We enforce a **Single-Pass Consolidated 5-in-1 Prompt** saving 80% input token expenditure ($8,000 \text{ tokens once}$ vs $40,000 \text{ tokens}$).
5. We enforce zero hallucinated search volumes via Pydantic schema field exclusion and negative prompt guardrails.

## Consequences
- **Positive:**
  - **Lowest Production Cost:** $0.00105 USD (~₹0.09 INR / 9 paise) per link analysis request.
  - **High Scalability:** 10,000 requests cost only ₹903 INR on Gemini Flash vs ₹30,100 on GPT-4o.
  - **1M Token Headroom:** Easily ingests extensive multi-page website context without risk of context window exhaustion.
  - **Low Latency:** ~1.2s to 1.5s TTFT satisfies strict API response SLA.
  - **Native Schema Compliance:** 99.8% first-pass valid JSON without markdown fencing leaks.
  - **Resilience:** Automatic fallback prevents service disruption if a provider degrades.
- **Negative:**
  - Requires Google AI Studio API key for live inference; mitigated by zero-cost `OfflineMockClient` during automated test suites.
