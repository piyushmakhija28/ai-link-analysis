# DevOps Architecture: Render.com Cloud Deployment Guide

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  

---

## 1. Deployment Specification (Dxora Section 11 Compliance)

Section 11 of the Dxora Technologies specification explicitly requires:
1. Deploy completed application on a publicly accessible server.
2. Provide a **Live Demo URL** for the deployed application.
3. Share the API documentation link (`/docs`) for testing and validation.
4. Ensure the endpoint works end-to-end and returns structured JSON.

We provide a zero-configuration Infrastructure-as-Code blueprint in `render.yaml` for 1-click continuous deployment from GitHub on Render.com.

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
    envVars:
      - key: APP_NAME
        value: "AI-Powered Website Link Analysis API"
      - key: APP_ENV
        value: "production"
      - key: PORT
        value: "4000"
      - key: LLM_MODEL
        value: "gemini-1.5-flash"
      - key: MAX_CONTENT_LENGTH
        value: "50000"
      - key: REQUEST_TIMEOUT
        value: "10"
      - key: CACHE_TTL
        value: "86400"
      - key: GEMINI_API_KEY
        sync: false
```

---

## 3. Deployment Verification Endpoints

| Target | URL Format | Expected Response |
|:---|:---|:---|
| **Live API Endpoint** | `https://ai-link-analysis.onrender.com/link-analysis` | HTTP 200 JSON with 5 BI sections |
| **Interactive Docs** | `https://ai-link-analysis.onrender.com/docs` | Swagger UI documentation |
| **SRE Health Check** | `https://ai-link-analysis.onrender.com/health` | HTTP 200 `{"status": "healthy"}` |
