# ADR-008: Correlation ID Tracing and RFC 7807 Problem Details

## Status
Accepted

## Context
In production microservice architectures, tracing errors and debugging distributed requests across web scrapers and external LLMs requires end-to-end correlation. Furthermore, unstructured error responses (e.g. plain strings or ad-hoc JSON) confuse client applications.

## Decision
1. **Correlation ID Tracing:** Implement ASGI middleware that inspects or generates a UUIDv4 `X-Correlation-ID`, propagating it through `contextvars` to all structured JSON logs, outbound client requests, and the final response header.
2. **RFC 7807 Problem Details:** All error responses return `application/problem+json` formatted payloads containing `type`, `title`, `status`, `detail`, `instance`, `correlation_id`, and `timestamp`.

## Consequences
- **Positive:**
  - Fast, unambiguous distributed debugging and log correlation.
  - Industry-standard error representation adhering to IETF RFC 7807.
- **Negative:**
  - Small overhead in context variable lookups and structured envelope formatting.
