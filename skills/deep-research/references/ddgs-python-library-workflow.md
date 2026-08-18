# DDGS Python Library Workflow for Deep Research

## When to Use This Approach
Use the `duckduckgo_search` Python library directly (via `execute_code`) when you need[cite: 2]:
* **Parallel queries:** Run multiple search queries simultaneously in a single Python script[cite: 2].
* **Fine-grained control:** Programmatically adjust result limits, regions, safe search parameters, and time filters[cite: 2].
* **Structured data extraction:** Parse and filter search payloads directly before launching headless browser navigation[cite: 2].

---

## Setup & Installation

```bash
pip install duckduckgo_search
```

---

## Implementation Patterns

### 1. Basic Multi-Query Parallel Batch

```python
from duckduckgo_search import DDGS

queries = [
    "primary entity architectural overview",
    "competitor benchmarks comparison",
    "security vulnerabilities limitations"
]

all_results = {}
with DDGS() as ddgs:
    for q in queries:
        results = list(ddgs.text(q, max_results=6))
        all_results[q] = [
            {"title": r.get("title"), "url": r.get("href"), "snippet": r.get("body")}
            for r in results
        ]

print(all_results)
```

### 2. Time-Filtered & Domain-Scoped Search

```python
from duckduckgo_search import DDGS

# timelimit options: 'd' (day), 'w' (week), 'm' (month), 'y' (year)
with DDGS() as ddgs:
    results = list(ddgs.text(
        "sustainable engineering standards site:gov OR site:org",
        timelimit="m",
        max_results=8
    ))

for item in results:
    print(f"- {item['title']}: {item['href']}")
```

---

## Result Structure Reference

Each result returned by `ddgs.text()` contains a dictionary with standard fields[cite: 2]:
* `title`: Page title[cite: 2]
* `href`: Target web URL[cite: 2]
* `body`: Snippet or summary text (frequently truncated)[cite: 2]

---

## Pipeline: DDGS Search + Deep Extraction

1. **Search with DDGS:** Execute 3–5 queries in parallel via code execution[cite: 2].
2. **Identify Authoritative Targets:** Isolate primary documentation, whitepapers, and regulatory links[cite: 2].
3. **Navigate & Extract:** Route selected URLs to `tavily_extract` (for clean markdown) or `browser_navigate` (for client-rendered applications)[cite: 2].
4. **Cross-Verification:** Execute targeted follow-up queries if conflicting data is encountered across sources[cite: 2].

---

## Pitfalls & Failure Handling

* **Rate Limiting / Throttling:** If empty results are returned, throttle calls, reduce batch size, or escalate directly to `tavily_search`[cite: 2].
* **Truncated Body Text:** DDGS body snippets are often shortened and should not be used alone for critical claims without visiting the primary source[cite: 2].
* **Generic Aggregator Results:** If queries return SEO aggregators or shallow listicles, use domain-targeted queries (`site:domain.com`) to query primary sources directly[cite: 2].

---

## When NOT to Use DDGS

* Simple, one-off factual queries (use primary search tools directly)[cite: 2].
* Real-time streaming news feeds requiring live timestamps[cite: 2].
* Deep JS-rendered single-page applications requiring full DOM interaction[cite: 2].
