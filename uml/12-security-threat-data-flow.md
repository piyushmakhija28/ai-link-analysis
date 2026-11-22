# UML 12: Security Threat & Data Flow Diagram

```mermaid
graph TD
    subgraph UntrustedZone["Untrusted External Zone"]
        Attacker["Potential Adversary"]
        MaliciousSite["Adversary-Controlled Website"]
    end

    subgraph DemilitarizedZone["Security Ingress Boundary"]
        IngressRouter["FastAPI Ingress Router"]
        SchemeCheck["Scheme Validator (Strict http/https)"]
        PortCheck["Port Validator (Strict 80, 443)"]
        DNSCheck["DNS Pre-Flight Resolver"]
        IPBlacklist["IP Blacklist Validator"]
    end

    subgraph InternalZone["Protected Microservice Runtime"]
        StreamFetcher["HTTPX Stream Reader (5MB Hard Cap)"]
        DOMSanitizer["BeautifulSoup DOM Pruner (Strip scripts/styles/nav)"]
        XMLFencer["Prompt Fencing Engine (untrusted_scraped_website_content)"]
        PydanticGuard["Pydantic Schema Validator & JSON Repair"]
    end

    subgraph ProtectedCloud["Protected Cloud / Local Infrastructure"]
        MetadataService["169.254.169.254 (Cloud Metadata)"]
        InternalServices["127.0.0.1, 10.0.0.0/8 (Internal VPC)"]
        GeminiAPI["Google Gemini 1.5 Flash API"]
    end

    Attacker -->|"SSRF Attack Payload"| IngressRouter
    IngressRouter --> SchemeCheck
    SchemeCheck --> PortCheck
    PortCheck --> DNSCheck
    DNSCheck --> IPBlacklist
    IPBlacklist --x|"BLOCKED: 400 SSRF Violation"| MetadataService
    IPBlacklist --x|"BLOCKED: 400 SSRF Violation"| InternalServices
    IPBlacklist -->|"Safe Public IP"| StreamFetcher
    StreamFetcher --> MaliciousSite
    MaliciousSite -.->|"Prompt Injection Payload"| StreamFetcher
    StreamFetcher --> DOMSanitizer
    DOMSanitizer --> XMLFencer
    XMLFencer --> GeminiAPI
    GeminiAPI --> PydanticGuard
```
