# ADR-003: Asynchronous HTTPX Client with Streaming Body Protection

## Status
Accepted

## Context
The microservice must fetch external website HTML content across arbitrary third-party servers. Upstream web servers may be slow, malicious, or serve massive files (multi-megabyte responses, zip bombs, slowloris streams).

Alternatives considered:
1. **Requests (Sync):** Blocks the Python event loop during network socket I/O, drastically reducing microservice throughput.
2. **AIOHTTP:** High performance, but more complex API and less straightforward integration with pytest mock transports.
3. **HTTPX (Async):** Modern HTTP client supporting both HTTP/1.1 and HTTP/2, connection pooling, standard timeout configurations, streaming response iterators, and first-class `MockTransport` support for unit testing.

## Decision
We select **HTTPX (AsyncClient)** with a shared connection pool, 10.0s total SLA timeout (5.0s connect, 10.0s read), standard desktop User-Agent header, and streaming byte-size counting aborting at 5MB.

## Consequences
- **Positive:**
  - Non-blocking socket I/O preserves ASGI worker throughput.
  - Streaming payload limit protects against memory exhaustion (OOM) and decompression bombs.
  - `MockTransport` enables fast, hermetic unit tests with zero network calls.
- **Negative:**
  - Streaming responses require explicit byte-counter loops rather than reading `.text` directly.
