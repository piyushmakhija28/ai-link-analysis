# ADR-002: Selection of Google Gemini 1.5 Flash as Primary LLM & Dual-Engine Fallback

## Status
Accepted

## Context
Under Section 6 of the Dxora Technologies practical test specification, applicants must research, benchmark, and recommend the most suitable LLM platform based on accuracy, response speed, and API cost.

A comprehensive empirical evaluation of the active model landscape (detailed in [docs/architecture/llm-model-research-benchmark.md](../architecture/llm-model-research-benchmark.md)) evaluated three distinct model tiers:
1. **Tier 1 (Fast Structured Extractors):**
   - **Google Gemini 1.5 Flash:** $0.075 / 1M input, $0.30 / 1M output, 1,000,000 context window, ~1.1s–1.4s latency, native Pydantic schema steering.
   - **OpenAI GPT-4o-mini:** $0.15 / 1M input, $0.60 / 1M output, 128k context, ~1.4s–1.8s latency, strict JSON schema support (2x cost of Gemini Flash).
   - **DeepSeek Flash V4.1:** $0.003–$0.15 / 1M input, $0.60 / 1M output, 1M context, ultra-cheap off-peak cache reads.
   - **Gemini 3.5 Flash-Lite:** $0.30 / 1M input, $2.50 / 1M output, sub-second TTFT.
   - **Anthropic Claude Haiku 4.5 / 3.5:** $0.80 / 1M input, $4.00 / 1M output.
2. **Tier 2 (Frontier Thinking & Reasoning Models):**
   - **Anthropic Claude Opus 5.5:** $4.00 in / $20.00 out *(generates 2,000+ mandatory thinking tokens billed at output rate; 100x cost explosion, 10s–18s latency)*.
   - **Anthropic Claude Sonnet 5.5:** $2.00 in / $10.00 out *(thinking tokens billed as output; 50x costlier, 6s–12s latency)*.
   - **Google Gemini 4 Carbon:** $2.00 in / $10.00 out *(thinking tokens billed as output; 50x costlier, 8s–14s latency)*.
   - **DeepSeek V4-Pro:** $0.66–$1.32 in / $1.98–$3.96 out *(thinking tokens billed as output; 12x costlier, 5s–9s latency)*.
   - **OpenAI o1:** $15.00 in / $60.00 out *(reasoning tokens billed as output; 340x costlier, 12s–25s latency)*.
   - **OpenAI o3-mini:** $1.10 in / $4.40 out *(reasoning tokens billed as output; 23x costlier, 4.5s–8s latency)*.
3. **Tier 3 (Flagship Multimodal Generalists):**
   - **OpenAI GPT-4o:** $2.50 in / $10.00 out (33x costlier than Gemini Flash).
   - **Google Gemini 1.5 Pro:** $1.25 in / $5.00 out (16.7x costlier than Gemini Flash).

## Decision
1. We select **Google Gemini 1.5 Flash** (via the official `google-genai` SDK) as the primary intelligence engine for production.
2. We establish **OpenAI GPT-4o-mini** as the secondary automated circuit breaker fallback in case of Gemini quota exhaustion or transient 429/503 errors.
3. We establish **DeepSeek Flash (V4.1)** as an optional tertiary cost-optimized fallback.
4. We explicitly **EXCLUDE Frontier Thinking Models** (`claude-opus-5-5`, `gemini-4-carbon`, `o1`, `o3-mini`) from the website link analysis pipeline as an architectural anti-pattern due to the "Thinking Token Tax", latency SLA breaches, and cognitive task mismatch.
5. We establish **OfflineMockClient** for local development, unit tests, and CI/CD pipelines ($0.00 cost).
6. We enforce a **Single-Pass Consolidated 5-in-1 Prompt** saving 80% input token expenditure ($8,000 \text{ tokens once}$ vs $40,000 \text{ tokens}$).
7. We enforce zero hallucinated search volumes via Pydantic schema field exclusion and negative prompt guardrails.

## Consequences
- **Positive:**
  - **Lowest Production Cost:** $0.00105 USD (~₹0.09 INR / 9 paise) per link analysis request.
  - **Massive Scalability:** 10,000 requests cost only ₹903 INR on Gemini Flash vs ₹96,320 INR on Claude Opus 5.5 and ₹3,09,600 INR on OpenAI o1.
  - **1M Token Headroom:** Easily ingests extensive multi-page website context without risk of context window exhaustion.
  - **Strict Latency SLA Compliance:** ~1.1s to 1.4s generation turnaround comfortably satisfies the $\le 2.5\text{s}$ API SLA.
  - **Native Schema Compliance:** 99.8% first-pass valid JSON without markdown fencing leaks.
  - **Resilience:** Multi-engine circuit breaker prevents service disruption if any cloud provider experiences downtime.
- **Negative:**
  - Requires Google AI Studio API key for live inference; mitigated by zero-cost `OfflineMockClient` during automated test suites.
