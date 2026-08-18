# Deep Research Agent Skill for Hermes

[![Agent Skill](https://img.shields.io/badge/Agent-Skill-blue.svg)](https://github.com/Zivatron/Agent-Zivatron)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

An agentic research pipeline for Hermes and MCP-compatible agents. It executes multi-source investigations by combining parallel free search via Python (`duckduckgo_search`), semantic verification via Tavily, and headless browser scraping for deep DOM extraction.

---

## 🏗 Directory Structure

```text
skills/deep-research/
├── SKILL.md                                  # Core agent instructions & pipeline definition
└── references/
    ├── ddgs-python-library-workflow.md       # Fast parallel query execution patterns
    └── curated-research-report-format.md     # Output templates for industry & policy briefings
```
⚡ Tool Execution Precedence
To maximize speed and minimize API costs, this skill enforces a strict waterfall model:

```Plaintext
[User Inquiry]
       │
       ▼
1. Free Breadth Discovery (DDGS / Python) ──► Instant parallel SERP queries via code execution
       │
       ▼
2. Semantic Depth & Extraction (Tavily)   ──► Clean markdown extraction & anomaly verification
       │
       ▼
3. Dynamic Browser Scraping (Headless)    ──► JavaScript/SPA rendering & fallback navigation
Credit Discipline: Caps Tavily API usage to a maximum of 50 calls per session before falling back to local scraping and DDGS.
```

Cross-Verification: Anomaly detection automatically triggers targeted disproof queries before claiming consensus.

## 📦 Prerequisites
Python Environment: Requires duckduckgo_search for parallel search batching:

Bash
pip install duckduckgo_search
Agent Capabilities:

Python Code Execution tool (execute_code)

Tavily MCP tools (tavily_search, tavily_extract, tavily_crawl) (Optional but recommended)

Headless Browser tools (browser_navigate, browser_snapshot, browser_click)

## 🚀 Installation
Clone the repository and copy the deep-research skill into your agent workspace:

Bash
git clone https://github.com/Zivatron/Agent-Zivatron.git
cp -r Agent-Zivatron/skills/deep-research ~/.hermes/skills/
If loading dynamically into an active agent session, reference SKILL.md directly.

## 🎯 Usage & Triggers
When It Triggers
Comprehensive multi-source industry, policy, or academic investigations.

Competitive landscape mapping and deep-dive technical comparisons.

Market trend reports requiring structured citations.

Anti-Triggers
Quick factual lookups or single-entity definitions.

Simple code debugging or routine single-source queries.

## 📄 Output Formats
Standard Synthesis Report: Executive summary, thematic analyses with inline citations ([[1]](URL)), discrepancy tables, and a verified source registry.

Curated Policy Briefing: Designed for institutional decision-makers, featuring 5 high-impact sources with relevance analysis and actionable organizational next steps (see references/curated-research-report-format.md).

🛡 Anti-Patterns & Safety Protocols
Zero Blind Clicking: browser_snapshot must always precede interactions to verify element IDs.

Aggregator Escape: When news aggregators return SEO spam, the agent pivots directly to root regulatory domains.

Source Diversity: Requires a minimum of 3 independent domains before synthesizing conclusions.

## 🤝 Contributing
Issues and PRs are welcome on GitHub.
