# ADR-007: Deterministic Offline Mock Fallback for Development, CI & Rate Limits

## Status
Accepted

## Context
Under Section 2, Section 7, and Section 9 of the Dxora Technologies specification:
1. Candidate applications must never commit sensitive API credentials to public repositories (OWASP Zero-Trust).
2. The application must degrade gracefully if third-party credentials are not supplied in the environment.
3. Automated test suites and CI/CD pipelines must execute deterministically without incurring API billing or relying on external network connectivity.

### Alternatives Considered:
1. **Hard Failure on Missing Credentials:** Crash the application or return HTTP 500 when `GEMINI_API_KEY` is missing. Unacceptable: causes evaluation runs and CI pipelines to fail if evaluators do not immediately supply their own API keys.
2. **VCR.py / Recorded HTTP Cassettes:** Works well for fixed test cases, but fragile when URLs or schemas change, and does not help live evaluation requests on unconfigured environments.
3. **Deterministic Rule-Based Offline Mock Client (`OfflineMockClient`):** A custom strategy implementing `BaseLLMClient` that parses extracted metadata (title, meta description, domain name) and algorithmically synthesizes valid, structured 5-dimension business intelligence payloads without network calls.

## Decision Drivers
- Zero-cost test execution in GitHub Actions CI pipelines.
- Smooth onboarding for Dxora evaluators without mandatory pre-configuration of API keys.
- High-fidelity synthetic payloads conforming 100% to Pydantic v2 schemas.

## Decision
We implement `OfflineMockClient` as a first-class strategy and default fallback when `GEMINI_API_KEY` is absent or rate-limited.

## Consequences
- **Positive:**
  - Automated test suite (200+ tests) runs in <5 seconds at **$0.00 API cost**.
  - Reviewers can clone the repo and execute `POST /link-analysis` out of the box without providing credentials.
  - Clear telemetry labeling (`provider: "offline-mock"`, `cost_usd: 0.0`) provides full diagnostic transparency.
- **Negative:**
  - Mock intelligence is derived algorithmically from extracted metadata rather than frontier generative reasoning.
