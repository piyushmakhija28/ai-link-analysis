# 🔗 AI-Powered Website Link Analysis API

> **Dxora Technologies – Python Practical Test Submission**
> **Candidate:** Piyush Makhija
> **Deadline:** 27th November 2026

---

## 📌 Project Overview

This project is an **AI-powered REST API** built with **FastAPI** that accepts any website URL, intelligently extracts its content, and returns a **structured business intelligence report** using an LLM (Large Language Model).

The API goes beyond simple scraping — it **understands** the website like a business analyst would, and generates insights across five strategic dimensions.

---

## 🧠 My Understanding of the Problem

### What is the core problem?
Companies want to quickly understand *any* website — their competitors, their own brand, or a client's website — from a business intelligence perspective. Manually doing this takes hours. This API automates it in seconds.

### What does this API solve?
Given a URL like `https://example.com`, the API:
1. **Fetches** the actual HTML content of the website
2. **Extracts** meaningful text — ignoring boilerplate, navbars, footers
3. **Understands** the business context using an LLM
4. **Returns** a structured JSON with 5 deep analysis sections

### Why is this hard?
- Websites have wildly different structures (SPAs, server-side rendered, JavaScript-heavy)
- LLMs need clean, relevant content — raw HTML is noisy
- Competitor identification requires real-world knowledge, not just what's on the page
- SEO analysis needs domain expertise baked into prompts
- Output must be **consistently structured JSON** — LLMs can hallucinate format

---

## 🏗️ Architecture & Design Decisions

### High-Level Flow

```
Client → POST /link-analysis
           ↓
     URL Validation (SSRF protection)
           ↓
     Website Content Extraction
     (httpx + BeautifulSoup)
           ↓
     Content Cleaning & Chunking
           ↓
     LLM Analysis (Gemini 1.5 Flash)
     [5 sections in structured prompt]
           ↓
     Response Validation (Pydantic)
           ↓
     Structured JSON Response
```

### Technology Stack

| Component | Technology | Reason |
|---|---|---|
| **Framework** | FastAPI | Async, fast, Swagger auto-docs, Pydantic built-in |
| **Server** | Uvicorn | ASGI server, production-grade |
| **HTTP Client** | HTTPX | Async HTTP, better than Requests for FastAPI |
| **HTML Parser** | BeautifulSoup4 | Robust, well-tested HTML extraction |
| **LLM** | Google Gemini 1.5 Flash | Best cost/accuracy ratio (see Section 6 below) |
| **Validation** | Pydantic v2 | Strict schema enforcement, type safety |
| **Config** | python-dotenv | Secure environment variable management |

---

## 📁 Project Structure

```
ai-link-analysis/
│
├── app/
│   ├── main.py              # FastAPI app, route definitions
│   ├── config.py            # Settings, env vars, constants
│   ├── schemas.py           # Pydantic request/response models
│   │
│   ├── services/
│   │   ├── website_extractor.py    # URL fetch + HTML parsing
│   │   ├── llm_service.py          # LLM API integration
│   │   ├── analysis_service.py     # Orchestrates 5-section analysis
│   │   └── competitor_service.py   # Competitor research logic
│   │
│   └── utils/
│       └── validators.py           # URL validation, SSRF protection
│
├── tests/
│   └── test_link_analysis.py       # Unit + integration tests
│
├── .env.example             # Template for environment variables
├── requirements.txt         # Python dependencies
└── README.md                # This file
```

---

## 🔌 API Specification

### Endpoint

```http
POST http://localhost:4000/link-analysis
```

### Request Body

```json
{
  "url": "https://example.com"
}
```

### Response Structure

```json
{
  "success": true,
  "website": "https://example.com",
  "brand": {
    "name": "Example Corp",
    "category": "Technology Services",
    "summary": "A company providing digital solutions for businesses."
  },
  "analysis": {
    "competitor_analysis": {
      "brand_overview": "...",
      "competitors": [
        {
          "name": "Competitor A",
          "strengths": ["..."],
          "weaknesses": ["..."]
        }
      ],
      "competitive_advantages": ["..."],
      "opportunities": ["..."]
    },
    "target_search_queries": [
      {
        "query": "best digital solutions company",
        "platform": "Google",
        "intent": "Service Discovery",
        "relevance": "...",
        "target_audience": "..."
      }
    ],
    "seo_improvements": {
      "primary_keyword": "...",
      "secondary_keywords": ["..."],
      "long_tail_keywords": ["..."],
      "recommended_title": "...",
      "meta_description": "...",
      "supporting_content": ["..."]
    },
    "top_services": [
      {
        "name": "...",
        "description": "...",
        "target_industry": "...",
        "target_customer": "...",
        "main_benefit": "..."
      }
    ],
    "target_audience": {
      "primary_audience": ["..."],
      "secondary_audience": ["..."],
      "industries": ["..."],
      "customer_needs": ["..."],
      "geographical_focus": "..."
    }
  }
}
```

---

## 🤖 LLM Research & Model Recommendation

### Model Comparison

| Criteria | OpenAI GPT-4o | Google Gemini 1.5 Flash | Anthropic Claude 3.5 Haiku |
|---|---|---|---|
| **Input Cost** | $2.50 / 1M tokens | **$0.075 / 1M tokens** | $0.80 / 1M tokens |
| **Output Cost** | $10.00 / 1M tokens | **$0.30 / 1M tokens** | $4.00 / 1M tokens |
| **Context Window** | 128K tokens | **1M tokens** | 200K tokens |
| **Response Speed** | ~3-5 sec | **~1-2 sec** | ~2-4 sec |
| **JSON Output** | ✅ Excellent | ✅ Excellent | ✅ Excellent |
| **Web Content Understanding** | ✅ Excellent | ✅ Excellent | ✅ Excellent |
| **Cost for ~10K tokens** | ~$0.125 | **~$0.004** | ~$0.048 |

### ✅ Recommended Model: **Google Gemini 1.5 Flash**

**Reasons:**
1. **~30x cheaper** than GPT-4o for the same task
2. **1M token context window** — can handle large websites without chunking
3. **Sub-2 second** response time for structured outputs
4. **Native JSON mode** — reliable structured output
5. **Google's infrastructure** — 99.9% uptime SLA

**Fallback Model:** `gemini-1.5-pro` for complex websites where Flash struggles

### 💰 Cost Estimation per Analysis

| Item | Tokens | Cost (Gemini Flash) |
|---|---|---|
| Input (extracted content + prompt) | ~8,000 tokens | ~$0.0006 |
| Output (5-section JSON response) | ~2,000 tokens | ~$0.0006 |
| **Total per request** | ~10,000 tokens | **~$0.0012** |

**That's less than $0.002 per website analysis!**

### 🔧 Cost Optimization Strategies

1. **Pre-clean HTML** — Strip scripts, styles, nav, footer before sending to LLM
2. **Smart content extraction** — Focus on `<main>`, `<article>`, `<h1-h3>`, meta tags
3. **Prompt optimization** — Request all 5 sections in a single API call (not 5 separate calls)
4. **Response caching** — Cache analysis for same URL (TTL: 24 hours)
5. **Model tiering** — Use Flash for most sites, Pro only for complex/large sites

---

## 🛡️ Security Considerations

- **SSRF Protection:** Block private IP ranges (10.x.x.x, 192.168.x.x, 127.0.0.1, localhost)
- **URL Validation:** Strict regex + domain allowlist approach
- **API Key Security:** All credentials via environment variables, never hardcoded
- **Rate Limiting:** Prevent abuse with request throttling
- **Timeout Handling:** Max 10 seconds for website fetching

---

## 🚀 Setup & Installation

### Prerequisites

- Python 3.10+
- Google Gemini API Key (free tier available at [ai.google.dev](https://ai.google.dev))

### Local Setup

```bash
# 1. Clone the repository
git clone https://github.com/piyushmakhija/ai-link-analysis.git
cd ai-link-analysis

# 2. Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure environment
cp .env.example .env
# Edit .env and add your GEMINI_API_KEY

# 5. Run the server
uvicorn app.main:app --host 0.0.0.0 --port 4000 --reload
```

### Environment Variables

```env
GEMINI_API_KEY=your_api_key_here
LLM_MODEL=gemini-1.5-flash
MAX_CONTENT_LENGTH=50000
REQUEST_TIMEOUT=10
CACHE_TTL=86400
```

### Testing the API

```bash
# Using curl
curl -X POST http://localhost:4000/link-analysis \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com"}'

# API Documentation (Swagger UI)
open http://localhost:4000/docs
```

---

## 🌐 Live Demo

> 🔗 **API Endpoint:** `https://ai-link-analysis.onrender.com/link-analysis`
> 📚 **API Docs:** `https://ai-link-analysis.onrender.com/docs`

*(Will be updated after deployment)*

---

## 📊 Evaluation Criteria Coverage

| Criteria | Status | Notes |
|---|---|---|
| FastAPI implementation & API structure | 🔄 In Progress | Clean route design, Pydantic schemas |
| Website extraction & content understanding | 🔄 In Progress | HTTPX + BeautifulSoup + content cleaning |
| LLM integration & structured output | 🔄 In Progress | Gemini 1.5 Flash with JSON mode |
| Competitor & search query analysis | 🔄 In Progress | 3+ competitors, 5 platform queries |
| SEO recommendations & keyword intelligence | 🔄 In Progress | Full keyword + title + meta + content plan |
| Services & target audience identification | 🔄 In Progress | 5 services, audience segmentation |
| LLM model research & cost optimization | ✅ Done | See LLM Comparison section above |
| Error handling, code quality & documentation | 🔄 In Progress | Pydantic validation, graceful errors |

---

## 👨‍💻 About the Developer

**Piyush Makhija**
- Full Stack Developer | Python & AI Enthusiast
- GitHub: [@piyushmakhija](https://github.com/piyushmakhija)

---

*Submission for Dxora Technologies Python Practical Test*
*Deadline: 27th November 2026*
