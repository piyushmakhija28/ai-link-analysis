# UML 11: Deployment Diagram - Cloud Native Container Topology

```mermaid
deploymentDiagram
    title Deployment Topology on Render.com & Local Docker

    node ClientNode as "Client Host / Evaluator Environment" {
        artifact CurlClient as "HTTP Client / Curl / Web Browser"
    }

    cloud CloudPlatform as "Render.com Cloud Infrastructure (Oregon Region)" {
        node LoadBalancer as "Render Cloud Edge Load Balancer" {
            artifact TLSProxy as "TLS 1.3 Termination & DDoS Mitigation"
        }

        node ContainerHost as "Linux Container Host (Debian Bookworm)" {
            node OCIContainer as "Docker Container (ai-link-analysis)" {
                artifact UserContext as "Unprivileged User (appuser:10001)"
                artifact UvicornWorker as "Uvicorn ASGI Worker Process (:4000)"
                artifact MemoryCache as "In-Memory SHA-256 Cache"
            }
        }
    }

    cloud GoogleCloud as "Google Cloud Platform" {
        node GeminiService as "Google Gemini API Service" {
            artifact GeminiModel as "Gemini 1.5 Flash Model Endpoint"
        }
    }

    cloud OpenAIService as "OpenAI Cloud Platform" {
        node OpenAIServiceNode as "OpenAI API Service" {
            artifact GPTMiniModel as "GPT-4o-mini Model Endpoint"
        }
    }

    cloud PublicWeb as "Target Public Internet" {
        node TargetWebServer as "Target Public Web Server" {
            artifact WebPage as "HTML / CSS / Schema.org Assets"
        }
    }

    CurlClient --> TLSProxy : "HTTPS POST /link-analysis"
    TLSProxy --> UvicornWorker : "Forward to :4000 (Internal Bridge)"
    UvicornWorker --> MemoryCache : "Query & Store (24h TTL)"
    UvicornWorker --> TargetWebServer : "Async Streaming GET (<=5MB, 10s SLA)"
    UvicornWorker --> GeminiModel : "Primary: Single-Pass Prompt (HTTPS REST)"
    UvicornWorker -.-> GPTMiniModel : "Fallback: Circuit Breaker Query (HTTPS REST)"
```
