# Low-Level Design: Prompt Engineering & Schema Contract

> **Author:** Piyush Makhija (Technical Lead / Senior Software Engineer)  
> **Client / Evaluator:** Dxora Technologies (`hr@chivoxai.com`)  
> **Version:** 1.0.0-PROD  
> **Date:** November 2026  

---

## 1. Prompt Engineering Architecture

The prompt engineering strategy uses **Single-Pass Multi-Section Synthesis** with **XML Boundary Fencing** to achieve maximum accuracy, speed, and cost efficiency.

### 1.1 Defensive XML Boundary Fencing
Scraped website text is untrusted input. The prompt wraps it in strict boundary fences:
```text
<untrusted_scraped_website_content>
{{ pruned_website_text }}
</untrusted_scraped_website_content>
```
System instructions strictly declare that text inside this fence is raw data to be analyzed, never executable instructions.

### 1.2 Zero-Hallucination Search Volume Negative Constraint
The prompt contains an explicit negative constraint:
> **MANDATORY RULE:** NEVER fabricate or hallucinate numerical search volumes (e.g., '10,000 searches/mo'). Output must focus on qualitative search intent, business relevance, and target audience alignment.

---

## 2. Production System Prompt Template

```text
You are an elite Senior Staff Business Intelligence and SEO Analyst.
Analyze the following website content enclosed in <untrusted_scraped_website_content> and generate a structured JSON report covering five strategic business intelligence sections.

INSTRUCTIONS:
1. Ground all analysis strictly in the provided website content. If information is inferred, clearly label it as an inference.
2. Section 1 (Competitor Analysis): Identify brand name, category, overview, and minimum 3 realistic direct or industry competitors with strengths, weaknesses, and your brand's competitive advantages.
3. Section 2 (Target Search Queries): Identify exactly 5 target search queries across Google, Facebook, Instagram, LinkedIn, and X with search intent, business relevance, and target audience. CRITICAL: Do NOT invent numerical search volumes.
4. Section 3 (SEO Improvements): Recommend primary, secondary, and long-tail keywords, an optimized SEO title, meta description (150-160 chars), and supporting landing page topics.
5. Section 4 (Top 5 Services): Extract 5 core services with description, target industry, customer type, and main business benefit.
6. Section 5 (Target Audience): Segment primary audience, secondary audience, industry verticals, customer profiles, and customer pain points.

OUTPUT FORMAT:
Return pure valid JSON conforming strictly to the required schema. Do not include markdown fences or explanatory text.

<untrusted_scraped_website_content>
{pruned_website_content}
</untrusted_scraped_website_content>
```
