# ADR-001: Selection of FastAPI as Core REST Framework

## Status
Accepted

## Context
The application requires a high-performance, asynchronous REST API microservice in Python capable of non-blocking I/O operations (concurrent HTTP website fetching and external LLM API communication), automatic OpenAPI documentation generation, and strict request/response data validation.

Alternatives considered:
1. **Flask (Sync / Werkzeug):** Traditional synchronous framework. Requires threading hacks or Celery for non-blocking operations, lacks native Pydantic v2 validation, and requires external plugins for OpenAPI/Swagger documentation.
2. **Django / Django REST Framework:** Heavy monolithic framework with ORM and admin overhead not required for a stateless link analysis microservice.
3. **FastAPI (ASGI):** Built natively on Starlette and Pydantic. Supports async/await out of the box, provides automatic OpenAPI 3.1.0 interactive documentation (`/docs`), and native type checking.

## Decision
We select **FastAPI 0.115+** running on **Uvicorn 0.34+** ASGI server.

## Consequences
- **Positive:**
  - High concurrency with native async/await for I/O-bound web extraction and LLM calls.
  - Native Pydantic v2 data validation with zero boilerplate.
  - Interactive Swagger UI (`/docs`) and ReDoc generated automatically.
  - Dependency Injection system (`Depends()`) enables seamless mock injection in tests.
- **Negative:**
  - Requires developers to understand ASGI event loops and avoid blocking CPU-bound calls.
