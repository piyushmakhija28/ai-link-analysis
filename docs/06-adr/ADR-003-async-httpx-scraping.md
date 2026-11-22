# ADR-003: Selection of Asynchronous HTTPX for Resilient Web Extraction

## Status
Accepted

## Context
Under Section 5 of the Dxora Technologies specification, the service must reliably fetch content from arbitrary public websites. Target servers vary wildly in response times, encoding standards, and payload sizes.

### Alternatives Considered:
1. **Python standard `urllib.request`:** Synchronous, blocking, lacks connection pooling, cumbersome redirect handling, and complex SSL context configuration.
2. **`requests` library:** Ubiquitous synchronous HTTP client. Blocks the Python event loop thread during network I/O, severely bottlenecking ASGI concurrency.
3. **`aiohttp` client:** Capable asynchronous HTTP client, but features a less ergonomic API, requires manual session lifecycle management, and lacks native support for HTTP/2.
4. **`httpx` (`httpx.AsyncClient`):** Modern, full-featured async HTTP client for Python 3. Supports HTTP/1.1 and HTTP/2, connection pooling, non-blocking streaming chunk iterators, granular timeout configurations, and official `MockTransport` support for unit testing without network sockets.

## Decision Drivers
- True async/await non-blocking network I/O matching FastAPI's event loop.
- Support for chunk-by-chunk streaming response consumption (`client.stream("GET", url)`), enabling immediate termination if a response exceeds 5MB before buffering into RAM.
- Granular timeout management (separate connect, read, write, and pool timeouts).
- Built-in `httpx.MockTransport` allowing 100% deterministic testing without mock-patching globals.

## Decision
We select **HTTPX 0.28+** (`httpx.AsyncClient`) with connection pooling limits (`max_connections=100`, `max_keepalive_connections=20`) and strict 10.0-second SLA timeouts.

## Consequences
- **Positive:**
  - Streaming reads prevent decompression bombs and memory exhaustion attacks.
  - Zero event loop blocking during external website fetching.
  - Standardized error hierarchy (`httpx.TimeoutException`, `httpx.HTTPStatusError`).
- **Negative:**
  - Client sessions must be properly managed via async context managers or application lifespan to avoid unclosed socket warnings.
