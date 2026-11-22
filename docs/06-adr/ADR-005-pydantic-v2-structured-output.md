# ADR-005: Pydantic v2 Schema Enforcement and JSON Repair Fallback

## Status
Accepted

## Context
LLMs are probabilistic generators. Even with JSON mode enabled, models may occasionally wrap output in markdown code fences (````json ... ````), omit optional fields, or experience minor JSON syntax deviations.

## Decision
We enforce strict **Pydantic v2 schemas** coupled with a deterministic **JSON extraction and repair loop**:
1. Google Gemini Flash is invoked with `response_mime_type="application/json"` and Pydantic schema guidance.
2. The raw response is cleansed of potential markdown formatting (`strip()`, regex fence removal).
3. `pydantic.TypeAdapter.validate_json()` validates the payload.
4. If validation encounters missing non-critical fields, a deterministic schema repair function applies default sensible values based on website title and headings rather than crashing the request.

## Consequences
- **Positive:**
  - 100% deterministic, type-safe API responses matching client contracts.
  - Resilience against minor LLM formatting anomalies.
- **Negative:**
  - Requires maintaining synchronized Pydantic models and schema validators.
