---
name: deep-research
description: Use when the user requests comprehensive research, multi-source investigation, market/academic analysis, or exhaustive technical synthesis. Do NOT use for quick factual lookups, single-source queries, or simple debugging tasks.
---

# Deep Research Agent

A structured pipeline for resolving complex, multi-faceted inquiries through parallel search execution, automated fact verification, primary source extraction, and cited synthesis.

## Search & Extraction Hierarchy

Always follow this tool precedence to optimize speed, reliability, and token/credit budget:

1. **Breadth & Discovery:** `execute_code` with `duckduckgo_search` (instant, free, parallelizable).
2. **Targeted Extraction:** `tavily_extract` (fast markdown conversion) or `tavily_search` (semantic depth/verification).
3. **Deep Traversal:** `tavily_crawl` (domain-level crawling) or targeted `browser_navigate`.
4. **Interactive Scraping (Fallback):** `browser_navigate` + `browser_snapshot` for dynamic SPAs or JS-rendered DOMs.

> **Budget Constraint:** Cap Tavily calls at **50 per session**. Track active API calls in your scratchpad. If the ceiling is reached, revert exclusively to DDGS and direct browser navigation.

---

## The Research Pipeline

### Phase 1: Decomposition & Query Generation
1. **Entity Mapping:** Extract core entities, historical context, key players, and adjacent technologies.
2. **Query Matrix:** Generate 3–5 targeted queries covering:
   - Primary definition / Core state
   - Competitive / Alternative ecosystem
   - Critical vulnerabilities, limitations, or counterarguments
   - Recent regulatory, policy, or market updates

### Phase 2: Live Extraction & Breadth
1. **Parallel API Search:** Run initial search batch via Python:
```python
from duckduckgo_search import DDGS

queries = ["query 1", "query 2", "query 3"]
with DDGS() as ddgs:
    results = {q: list(ddgs.text(q, max_results=5)) for q in queries}
print(results)
```
2. **Source Escalation:**
   - If DDGS is rate-limited: Fall back to `tavily_search`.
   - If search aggregators return generic SEO fluff: Bypass search engines and navigate directly to primary domain root URLs (e.g., official docs, governing bodies, regulatory portals).
3. **Content Ingestion:**
   - Use `tavily_extract` for high-signal URLs.
   - For pages requiring authentication, complex interactive state, or pagination: Use `browser_navigate` → `browser_snapshot` → `browser_click`.
   - Always run `browser_snapshot` to resolve element IDs before clicking. Never blind-click.

### Phase 3: Cross-Verification & Consensus
1. **Flag Anomaly Data:** Identify single-source claims, statistical anomalies, or dated metrics.
2. **Targeted Disproof:** Execute a targeted verification query aimed specifically at debunking or confirming the anomalous claim.
3. **Handle Disagreement:** If reputable sources conflict, document the discrepancy with explicit attribution to both parties. Do not declare an unverified consensus.

### Phase 4: Structured Synthesis
Format the final delivery using this standard layout:

```text
# [Research Topic]: Investigation & Synthesis

## Executive Summary
- [Key Finding 1]
- [Key Finding 2]
- [Key Finding 3]

## Comprehensive Analysis
### [Theme 1: Technical / Core Architecture]
Analysis with inline citations [[1]](URL).

### [Theme 2: Market / Regulatory Dynamics]
Analysis with inline citations [[2]](URL).

## Discrepancies & Verification Gaps
- Identified conflicts between sources or unverified claims.

## Sources & Reference Registry
1. [Source Title / Domain](URL) - Extracted [Date]
2. [Source Title / Domain](URL) - Extracted [Date]
```

---

## Anti-Patterns & Failure Recovery

| Failure Mode | Root Cause | Recovery Protocol |
| :--- | :--- | :--- |
| **SEO Loop / Generic Results** | News aggregators returning irrelevant aggregate articles. | Abandon aggregator queries. Pivot directly to tier-1 primary sources (governing standards, official docs, trade associations). |
| **DOM / Snapshot Staleness** | Headless browser reloading cached state or navigation loops. | Take a new `browser_snapshot`. If identical after 2 attempts, terminate session and extract via `tavily_extract`. |
| **Bot Blocking (403/CAPTCHA)** | Target site blocking headless automation. | Do not attempt bypass loops. Drop URL immediately and use search cache or secondary mirror sources. |
| **Single-Source Bias** | Synthesizing output from a single domain. | Enforce a minimum of **3 independent domains** before proceeding to synthesis. |

## External References
- See `references/ddgs-python-library-workflow.md` for parallelization scripts.
- See `references/curated-research-report-format.md` for specialized industry-curation output templates.


