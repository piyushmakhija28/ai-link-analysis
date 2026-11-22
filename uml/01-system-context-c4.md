# UML 01: C4 System Context Diagram

```mermaid
C4Context
    title System Context Diagram for AI-Powered Link Analysis API

    Person(user, "API Consumer", "Developer, SEO Specialist, CRM System, or Dxora Evaluator")
    System(link_api, "ai-link-analysis API", "FastAPI microservice extracting web content, executing SSRF defenses, and generating 5-section BI.")
    System_Ext(dns, "Public DNS Resolver", "Resolves target domain names for pre-flight SSRF subnet validation.")
    System_Ext(target_web, "Target Website Server", "Public HTTP/HTTPS web server hosting target landing page content.")
    System_Ext(gemini, "Google Gemini 1.5 Flash API", "Primary generative AI engine providing structured JSON intelligence via native schema steering.")
    System_Ext(openai, "OpenAI API (GPT-4o-mini)", "Secondary automated circuit-breaker fallback for high availability.")
    System_Ext(mock, "Offline Mock Engine", "Deterministic rule-based fallback for unconfigured environments and zero-cost testing.")

    Rel(user, link_api, "POST /link-analysis (JSON URL)", "HTTPS / JSON")
    Rel(link_api, dns, "Pre-flight hostname resolution", "DNS UDP/53")
    Rel(link_api, target_web, "Streams HTML content (<=5MB, 10s SLA)", "HTTP / HTTPS")
    Rel(link_api, gemini, "Single-pass 5-in-1 structured prompt (<8k tokens)", "HTTPS REST / gRPC")
    Rel_Back(gemini, link_api, "Validated 5-section JSON payload", "JSON")
    Rel(link_api, openai, "Tripped circuit breaker fallback query", "HTTPS REST")
    Rel(link_api, mock, "Offline diagnostic synthesis when unconfigured", "In-Memory")
    Rel(link_api, user, "Returns 5-section BI dossier + Telemetry", "JSON / HTTPS")
```
