# ADR-008: Correlation ID Distributed Tracing & RFC 7807 Problem Details

## Status
Accepted

## Context
Under Section 8 of the Dxora Technologies specification, API errors must provide clear diagnostic details to client applications without exposing internal stack traces or database connection strings. In distributed microservice architectures, tracking requests across asynchronous boundaries (router, DNS resolver, scraping client, LLM provider) requires deterministic correlation identifiers.

### Alternatives Considered:
1. **Ad-hoc Custom JSON Error Envelopes (`{"error": "string"}`):** Inconsistent across endpoints; lacks standard machine-readable error codes or remediation references.
2. **OpenTelemetry Full Agent Tracing:** Full Jaeger/Zipkin tracing infrastructure is valuable for 50-microservice meshes, but introduces excessive dependency overhead for a single container service.
3. **RFC 7807 Problem Details (`application/problem+json`) with Asynchronous Contextvars Correlation Tracing:** Standard IETF specification defining standardized fields: `type`, `title`, `status`, `detail`, `instance`, and custom extensions (`correlation_id`).

## Decision Drivers
- Strict compliance with RFC 7807 industry standard for HTTP APIs.
- End-to-end request tracing via `X-Correlation-ID` header and Python `contextvars`.
- Safe exception masking preventing internal stack trace leaks to API consumers.

## Decision
We adopt **RFC 7807 Problem Details** for all HTTP error responses and inject a UUID4 correlation ID into every request context via custom ASGI middleware.

## Consequences
- **Positive:**
  - Machine-readable error codes (`ERR_SSRF_BLOCKED`, `ERR_UPSTREAM_TIMEOUT`) allow client applications to programmatically handle failures.
  - Correlation ID is returned in both response headers and body, linking client bug reports directly to structured JSON logs.
  - 100% compliant with enterprise API design best practices.
- **Negative:**
  - Requires custom exception handler mapping in FastAPI to override default Starlette validation error responses.
