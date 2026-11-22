# ADR-007: Deterministic Offline Mock Client for Zero-Credential Evaluation

## Status
Accepted

## Context
Technical evaluators or automated CI/CD runners may execute the application without configuring a paid `GEMINI_API_KEY`, or third-party quota limits (HTTP 429) might be reached during testing. Crashing with an unhandled exception or 500 error creates an unacceptable developer experience.

## Decision
We implement an **Offline Mock Adapter** (`OfflineMockClient`) via the **Factory Method Pattern**:
1. When `GEMINI_API_KEY` is empty, invalid, or upstream returns HTTP 429 rate limits, the service transparently switches to the mock adapter.
2. The mock adapter uses extracted semantic metadata (title, headings, meta tags) and deterministic heuristics to generate a realistic, high-fidelity 5-section business intelligence report.
3. The response clearly signals mock execution in response telemetry: `"ai_model": "mock-diagnostic-fallback"`.

## Consequences
- **Positive:**
  - Evaluators can inspect, test, and run the API end-to-end without needing an active Google Cloud billing account.
  - 100% deterministic, hermetic test execution with zero external network dependency.
  - Graceful degradation under third-party LLM outages.
- **Negative:**
  - Requires maintaining mock generation logic aligned with the production Pydantic schema.
