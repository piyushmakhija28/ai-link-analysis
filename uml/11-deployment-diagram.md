# UML 11: Deployment Diagram

```mermaid
deployment
    title Deployment Architecture on Render.com Cloud Platform

    node RenderCloud as "Render.com Cloud (Oregon Region)" {
        node LoadBalancer as "Render Managed TLS Edge / Proxy" {
            artifact SSL as "Automatic Let's Encrypt SSL/TLS Certificate"
        }

        node DockerContainer as "Docker Container (non-root appuser:10001)" {
            artifact UvicornServer as "Uvicorn ASGI Server (:4000)"
            artifact FastAPIApp as "FastAPI Application (Python 3.13)"
            artifact CacheMem as "In-Memory LRU Cache Store"
        }
    }

    node ExternalWeb as "Internet / Public Web" {
        node TargetServers as "Target Public Websites (:80, :443)"
    }

    node GoogleCloud as "Google Cloud Platform" {
        node GeminiService as "Gemini 1.5 Flash Cloud API"
    }

    node ClientNode as "Client Node" {
        artifact CurlPostman as "HTTP Client (Browser, Curl, Evaluator)"
    }

    ClientNode --> LoadBalancer : "HTTPS / 443"
    LoadBalancer --> UvicornServer : "HTTP / 4000 (Internal Bridge)"
    UvicornServer --> FastAPIApp : "ASGI Event Loop"
    FastAPIApp --> CacheMem : "Read / Write"
    FastAPIApp --> TargetServers : "Outbound HTTP/HTTPS (<5MB Stream)"
    FastAPIApp --> GeminiService : "Outbound HTTPS REST (<8k Tokens)"
```
