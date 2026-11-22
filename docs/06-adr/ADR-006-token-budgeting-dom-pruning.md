# ADR-006: Multi-Stage Semantic DOM Pruning and Token Budgeting

## Status
Accepted

## Context
Modern web pages frequently span 100KB to 2MB in raw HTML, loaded with CSS styles, client-side JavaScript bundles, SVG graphics, navigation headers, footers, and cookie banners. Directly feeding raw HTML to an LLM:
- Inflates token costs by 10x–20x.
- Increases LLM processing latency (TTFT).
- Dilutes semantic signal, causing hallucinated or lower-quality business analysis.

## Decision
We implement a **Multi-Stage Semantic DOM Pruning and Token Budgeting Pipeline** via BeautifulSoup4:
1. **Tag Stripping:** Remove `<script>`, `<style>`, `<noscript>`, `<svg>`, `<canvas>`, `<iframe>`, cookie banners, and modal dialogs.
2. **Semantic Extraction:** Isolate `<title>`, `<meta name="description">`, OpenGraph tags (`og:*`), schema.org JSON-LD, headings (`<h1>`-`<h3>`), and `<main>`/`<article>` paragraphs.
3. **Budget Capping:** Enforce a hard ceiling of 32,000 characters (~8,000 tokens), preserving headings and core paragraphs.
4. **Single-Pass Prompt:** Consolidate all 5 analysis sections into a single query, cutting input token usage by 80% compared to 5 separate calls.

## Consequences
- **Positive:**
  - Cuts API costs down to ~$0.00105 per website analysis.
  - Sub-2s LLM generation speed.
  - Maximizes semantic clarity for competitor and SEO extraction.
- **Negative:**
  - Very heavy Single Page Applications (SPAs) relying purely on client-side JS rendering might yield minimal server-side HTML text. (Mitigated by extracting meta tags and title).
