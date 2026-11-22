# ADR-006: Multi-Stage Semantic DOM Pruning & Character Budgeting

## Status
Accepted

## Context
Under Section 5 and Section 6 of the Dxora Technologies specification, raw website HTML must be cleaned and compressed before being processed by an LLM. Enterprise websites often contain 100KB to 2MB of HTML, 80% to 90% of which consists of active scripts, CSS styles, SVGs, base64 images, cookie consent banners, and boilerplate navigation links.

Sending raw HTML directly to an LLM wastes thousands of tokens, inflates API bills, dilutes semantic focus, and increases prompt injection attack surface.

### Alternatives Considered:
1. **Raw HTML Ingestion:** Ingest raw HTML directly into Gemini's 1M context. Feasible, but inflates input token costs by 5x–10x and slows down inference latency.
2. **Plain Regex Tag Stripping (`re.sub('<.*?>', '', html)`):** Fast, but strips semantic structure, turns text into an unreadable blob, and preserves script contents if regex fails on nested tags.
3. **Headless Browser Rendering (Playwright / Puppeteer):** Handles client-rendered SPAs, but introduces massive container overhead (>500MB Chromium binaries), high RAM consumption, and increases per-request latency from 1s to 6s.
4. **BeautifulSoup4 / lxml DOM Traversal & Budgeting:** Strips active noise tags, isolates high-signal elements (`<title>`, `<meta>`, `<h1>`-`<h3>`, `<main>`, `<article>`), and applies strict character ceilings in <15ms of local CPU time.

## Decision Drivers
- High execution speed (<20ms in RAM).
- Zero-cost token reduction (85% byte reduction).
- Preservation of semantic hierarchy (headings, descriptions, structured data).
- Memory-safe character budget capping.

## Decision
We select **BeautifulSoup4** (with `lxml` parser) to execute multi-stage semantic pruning, capping extracted text at **32,000 characters (~8,000 tokens)**.

## Consequences
- **Positive:**
  - Reduces raw HTML payloads by 85% at $0.00 token cost.
  - Keeps prompt sizes well within the optimal 8,000-token pricing tier ($0.00060 input cost).
  - Strips inline scripts that could contain prompt injection payloads.
- **Negative:**
  - Does not execute client-side JavaScript for pure SPAs without server-rendered HTML; mitigated by extracting `<head>` OpenGraph and JSON-LD schema tags.
