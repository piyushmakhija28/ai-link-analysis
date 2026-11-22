# ADR-002: Selection of Google Gemini 1.5 Flash as Primary LLM

## Status
Accepted

## Context
Under Section 6 of the Dxora Technologies practical test specification, applicants must research, benchmark, and recommend the most suitable LLM platform based on accuracy, response speed, and API cost.

Evaluated Models:
1. **OpenAI GPT-4o:** High reasoning accuracy, but expensive ($2.50 / 1M input, $10.00 / 1M output) with ~3.8s latency.
2. **Anthropic Claude 3.5 Haiku:** Fast (~2.5s) and good accuracy, but priced at $0.80 / 1M input and $4.00 / 1M output (~8x more expensive than Gemini Flash).
3. **Google Gemini 1.5 Flash:** Massive 1,000,000 token context window, sub-2s average latency, native JSON schema mode, and lowest pricing ($0.075 / 1M input, $0.30 / 1M output).

## Decision
We select **Google Gemini 1.5 Flash** (via the official `google-genai` SDK) as the primary intelligence engine, with fallback to `gemini-1.5-pro` or a deterministic offline mock.

## Consequences
- **Positive:**
  - **33x cheaper** than GPT-4o for website business intelligence (~$0.00105 per analysis).
  - 1M context window effortlessly ingests large website summaries without truncation risk.
  - Sub-2s response speed satisfies user SLA requirements.
  - Native JSON schema mode enforces exact Pydantic v2 compliance without prompt hallucination.
- **Negative:**
  - Requires active Google AI Studio API key for live inference; mitigated by implementing a deterministic offline mock client for environments without billing.
