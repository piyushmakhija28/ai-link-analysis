# ADR-001: Selection of FastAPI as Core REST Microservice Framework

## Status
Accepted

## Context
Under Section 2 of the Dxora Technologies practical test specification, the service must expose a clean, performant REST API endpoint (`POST /link-analysis`) running on `localhost:4000`. The service is predominantly I/O-bound: it must resolve DNS asynchronously, stream external website HTML, and invoke cloud LLM endpoints over HTTPS.

### Alternatives Considered:
1. **Flask (Sync WSGI):** Traditional Python web framework. Synchronous by design; requires threading hacks, gevent monkey-patching, or Celery workers for non-blocking I/O. Lacks native Pydantic v2 validation and requires third-party extensions for OpenAPI/Swagger contracts.
2. **Django / Django REST Framework (DRF):** Heavyweight enterprise monolithic framework. Includes ORM, database migrations, authentication sessions, and admin portals that represent unnecessary overhead for a stateless, high-throughput microservice.
3. **Tornado / Sanic:** Asynchronous frameworks, but lack the vast ecosystem, community adoption, and deep Pydantic integration offered by FastAPI.
4. **FastAPI (ASGI / Starlette):** Native asynchronous framework built on Starlette and Pydantic v2. Provides first-class async/await concurrency, automatic OpenAPI 3.1.0 contract generation (`/docs`), strict type enforcement, and dependency injection.

## Decision Drivers
- High-concurrency throughput with native Python `asyncio` event loop.
- First-class Pydantic v2 integration for declarative request/response serialization.
- Built-in OpenAPI 3.1.0 and interactive Swagger UI documentation at zero development cost.
- Native Dependency Injection (`Depends()`) enabling seamless test mock substitution.

## Decision
We select **FastAPI 0.115+** running on the **Uvicorn 0.34+** ASGI server.

## Consequences
- **Positive:**
  - Non-blocking I/O concurrency allows a single worker process to handle hundreds of concurrent web scraping and LLM synthesis operations.
  - Automatic JSON request deserialization and RFC 7807 validation error generation.
  - Direct compilation of OpenAPI 3.1.0 specs aligning with Dxora evaluation requirements.
- **Negative:**
  - Requires developers to strictly avoid synchronous, blocking calls (e.g. `time.sleep()`, standard `requests`, or raw `socket.getaddrinfo()`) on the main event loop thread; blocking calls must be explicitly offloaded to thread pools (`run_in_executor`).
