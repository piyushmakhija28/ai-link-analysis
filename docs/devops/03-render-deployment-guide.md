# DevOps Architecture: Render.com Cloud Deployment & Live Verification Guide

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Repository:** [https://github.com/piyushmakhija28/ai-link-analysis](https://github.com/piyushmakhija28/ai-link-analysis)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  
> **Status:** Baselined & Enforced  

---

## 1. Compliance with Dxora Section 11 Deployment Specification

Section 11 of the Dxora Technologies practical assessment brief explicitly requires:
1. Deploy the completed application on a publicly accessible cloud server.
2. Provide a **Live Demo URL** for the running application.
3. Share the interactive API documentation link (`/docs`) for live testing and schema validation.
4. Ensure the endpoint works end-to-end and returns structured JSON responses matching Section 4.

We provide a zero-configuration **Infrastructure-as-Code (IaC)** blueprint in `render.yaml` for 1-click continuous deployment from GitHub to Render.com.

---

## 2. Infrastructure-as-Code Blueprint (`render.yaml`)

```yaml
services:
  - type: web
    name: ai-link-analysis
    env: docker
    dockerfilePath: Dockerfile
    plan: free
    region: oregon
    branch: main
    healthCheckPath: /health
    autoDeploy: true
    envVars:
      - key: APP_NAME
        value: "AI-Powered Website Link Analysis API"
      - key: APP_ENV
        value: "production"
      - key: PORT
        value: "4000"
      - key: PRIMARY_LLM_MODEL
        value: "gemini-1.5-flash"
      - key: FALLBACK_LLM_MODEL
        value: "gpt-4o-mini"
      - key: MAX_CONTENT_LENGTH
        value: "5242880"
      - key: REQUEST_TIMEOUT_SECONDS
        value: "10.0"
      - key: CACHE_TTL_SECONDS
        value: "86400"
      - key: LOG_LEVEL
        value: "INFO"
      - key: GEMINI_API_KEY
        sync: false
      - key: OPENAI_API_KEY
        sync: false
```

---

## 3. Deployment Verification Endpoints & Live Testing

| Interface | Public URL Target | Expected Response & Verification |
|:---|:---|:---|
| **Live API Endpoint** | `https://ai-link-analysis.onrender.com/link-analysis` | `POST` with `{"url": "https://dxora.com"}` $\to$ Returns HTTP 200 with 5 BI sections |
| **Interactive Docs** | `https://ai-link-analysis.onrender.com/docs` | OpenAPI 3.1.0 Swagger UI interactive console |
| **Alternative Docs** | `https://ai-link-analysis.onrender.com/redoc` | Clean ReDoc API documentation |
| **SRE Health Check** | `https://ai-link-analysis.onrender.com/health` | HTTP 200 `{"status": "healthy", "service": "ai-link-analysis"}` |
| **SRE Readiness Check**| `https://ai-link-analysis.onrender.com/ready` | HTTP 200 `{"status": "ready", "llm_provider": "google-gemini"}` |

### Live Curl Verification Command:
```bash
curl -X POST "https://ai-link-analysis.onrender.com/link-analysis" \
     -H "Content-Type: application/json" \
     -d '{"url": "https://dxora.com"}'
```

---
*End of Render.com Cloud Deployment Guide. Authored by Piyush Makhija for Dxora Technologies Evaluation.*
