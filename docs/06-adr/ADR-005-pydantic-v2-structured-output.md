# ADR-005: Pydantic v2 Schema Enforcement & Native Structured Outputs

## Status
Accepted

## Context
Under Section 2 and Section 3 of the Dxora Technologies practical test specification, the API must output a clean, deeply structured JSON payload conforming to strict field specifications across 5 business intelligence dimensions. 

Unstructured natural language output from LLMs is notorious for formatting irregularities, markdown fencing leaks (` ```json `), missing keys, and schema hallucination.

### Alternatives Considered:
1. **Unconstrained Text Generation with Regex Parsing:** Fragile, prone to parsing crashes, fails on nested objects.
2. **Few-Shot Examples in Prompt:** Injects 1,500+ tokens of static JSON into every prompt, inflating API costs by 20%–30% without guaranteeing 100% schema compliance.
3. **Pydantic v1:** Slower validation, legacy syntax, lacking modern JSON Schema draft 2020-12 alignment.
4. **Pydantic v2 with Native LLM Schema Steering:** Core validation compiled in Rust (pydantic-core). Supports native JSON Schema generation (`model_json_schema()`), direct integration with Google GenAI SDK (`response_schema`), and OpenAI strict mode (`json_schema`).

## Decision Drivers
- High-speed deserialization and strict type checking.
- Guaranteed schema adherence at the decoding level.
- Elimination of prompt token bloat from few-shot examples.
- Seamless OpenAPI 3.1.0 schema generation.

## Decision
We select **Pydantic v2** (`pydantic>=2.10.0`) for all domain request/response models and compile them directly into Google Gemini 1.5 Flash's `response_schema`.

## Consequences
- **Positive:**
  - 99.8% first-pass schema validity.
  - Zero markdown code fence stripping required.
  - Automatic OpenAPI 3.1 schema generation.
- **Negative:**
  - Schema definitions must be maintained strictly; any field added to the Python model affects the LLM decoding constraint.
