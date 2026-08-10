---
name: librarian
description: Use when adding, organizing, querying, or standardizing documents in your local or shared business document repository — including frontmatter enhancement, INDEX.md maintenance, RAG-based retrieval, and document library architecture.
author: Community
version: 2.0.0
---

# Installation & Setup Configuration

> **Note for Users Installing This Skill:**  
> Before running the Librarian skill, ensure you configure the following directory paths and organization settings:
>
> 1. **Document Directory Path:** Replace `/path/to/documents/` with your local or server absolute path (e.g., `/Users/username/documents/` or `/var/data/documents/`). **Always use absolute paths** so your agent resolves the correct folder regardless of its working directory.
> 2. **Organization / Company Name:** Set your default organization name in frontmatter templates (`company: Your Company Name`).
> 3. **Category Mappings:** Customize file naming prefixes (e.g., `SOP_*`, `POLICY_*`, `CONTRACT_*`) to match your organization's document classification schema.

---

# SKILL DEFINITION: LIBRARIAN

Design, store, and index document databases for AI agent reference. The **Index** is the structured map that lets the agent find and load the right document for any query — frontmatter tags, directory routing, and RAG (Retrieval-Augmented Generation) retrieval are its pillars.

## 1. THE INDEX PIPELINE SEQUENCE

When processing, ingesting, or maintaining the document library, execute these steps sequentially:

```
[Scan Frontmatter] ──> [Standardize Metadata] ──> [Directory Routing] ──> [Build INDEX.md] ──> [Apply RAG Protocol]
```

---

## 2. STEP-BY-STEP WORKFLOW

### Step 1 — Scan for Frontmatter
- **Scan Depth:** Read at least **2,000+ characters** per file (not just the first few lines). Header metadata, tags, or keyword blocks can push YAML frontmatter beyond 900+ characters; reading only 300 characters falsely reports "missing frontmatter" and cuts off the closing `---` delimiter.
- **Parsing Strategy:** Detect YAML frontmatter starting with the initial `---` delimiter.
- **Regex Parsing Rule:** Use quoted-value matching regex to parse titles and fields containing colons or special characters:
  ```regex
  ^title:\s*"([^"]*)"
  ```
  *(Unquoted regex breaks on titles containing dashes, colons, or parentheses, e.g., `"ISO 9001:2015 — Quality Management Systems"`).*

---

### Step 2 — Standardize Frontmatter
All managed documents must adhere to a standardized YAML structure and stay under **1,000 characters total**. If existing frontmatter exceeds 1,000 characters, condense non-essential fields.

#### Universal Schema (Maximum 6 Fields)
```yaml
---
title: "Human-Readable Document Title"
category: <type>
company: "Your Organization Name"
source: internal | external
status: active | archived
verification_status: <verified by Name | unverified — reason>
---
```

#### Field Rules & Normalization
1. **Field Cap:** Maximum **6 fields**. `verification_status` MUST be the 6th and final field.
2. **Always Quote Values:** Enclose `title` and string values in double quotes (`"..."`) to safely support colons, dashes, and parentheses without breaking YAML syntax.
3. **Exclude Metadata Bloat:** Never include verbose keyword lists, full descriptions, or department trees in frontmatter—keep it lean for fast query indexing.
4. **Derive Category from Prefix:** Automatically assign `category` when missing based on file naming conventions:
   - `STD_*` / `AS_*` / `ISO_*` → `standard`
   - `CODE_*` / `REG_*` → `code`
   - `SOP_*` → `sop`
   - `POLICY_*` → `policy`
   - `CONTRACT_*` / `LEGAL_*` → `legal`
   - `PROJ_*` → `project`
5. **Normalize Field Names:** Handle legacy or alternative key names during ingestion:
   - Map `document:` → `title:`
   - Normalize quoted (`category: "policy"`) and unquoted (`category: policy`) values.

#### Standardized Category Examples

##### Standards & Compliance
```yaml
---
title: "ISO 9001:2015 — Quality Management Systems Requirements"
category: standard
company: "Your Organization Name"
source: external
status: active
verification_status: unverified — external source document; cross-reference original PDF for exact specifications
---
```

##### Legal & Regulatory
```yaml
---
title: "Master Services Agreement — Client Vendor Framework 2026"
category: legal
company: "Your Organization Name"
source: internal
status: active
verification_status: verified by Legal Counsel
---
```

##### Internal Operations (Policies & SOPs)
```yaml
---
title: "SOP-014: Incident Reporting and Risk Assessment"
category: sop
company: "Your Organization Name"
source: internal
status: active
verification_status: verified by Operations Manager
---
```

---

### Step 3 — Organize Directory Structure
Store all documents under `/path/to/documents/`. **Always use absolute paths.** Never use relative paths like `workspace/documents/` because agent runtime working directories can vary, causing files to be created or searched in incorrect subfolders.

```text
/path/to/documents/
├── Organization/               (Business and operational documents)
│   ├── codes-and-regulations/  (Building/industry codes, government regulations)
│   ├── standards/              (ISO, industry, and technical specifications)
│   ├── legal/                  (Contracts, compliance, agreements)
│   ├── internal-policies/      (WHS, HR policies, SOPs, SWIs)
│   └── projects/               (Project-specific client and delivery records)
└── Archives/                   (Historical or deprecated records)
```

> **Strict Isolation Rule:** Separate business/operational documents from external or personal reference materials. Never mix confidential corporate files with general reference items in queries or automated summaries.

---

### Step 4 — Build `INDEX.md`
Parse frontmatter from every managed file and generate a category-by-category index table at `/path/to/documents/INDEX.md` (root level of the documents folder). 

- Keep `INDEX.md` lightweight (~2KB to 10KB).
- Load `INDEX.md` on-demand when a user mentions broad document search or structure queries, rather than pre-loading it into every conversation session.

#### Example `INDEX.md` Table Structure
```markdown
# Document Library Index

## Internal SOPs & Policies
| Title | File Path | Status | Verification Status |
| :--- | :--- | :--- | :--- |
| **SOP-014: Incident Reporting** | `Organization/internal-policies/SOP_014.md` | Active | Verified |

## External Standards & Regulatory
| Title | File Path | Status | Verification Status |
| :--- | :--- | :--- | :--- |
| **ISO 9001:2015 Quality Management** | `Organization/standards/STD_ISO_9001.md` | Active | Unverified |
```

---

### Step 5 — Apply RAG Retrieval Protocol

When answering questions involving legal contracts, safety standards, regulatory codes, or technical specifications:

1. **Load Full Text:** Read the complete document context—do not rely solely on frontmatter metadata or short summaries.
2. **Exact Citation:** Quote specific clause numbers, section headers, and paragraph identifiers verbatim. Never paraphrase legal, safety, or regulatory mandates.
3. **Source Attribution:** Cite the exact document title (from frontmatter `title`), absolute or relative repository file path, and clause references.
4. **Enforce Verification Status Disclaimers:**
   - **If `verified`:** Deliver answers directly with full confidence.
   - **If `unverified`:** Include an explicit disclaimer noting that the source document was converted from external formats (e.g., PDF) and advise cross-referencing the primary original document when exact precision is required.

#### When Full RAG Retrieval is NOT Required
- General questions regarding document layout, folder organization, or indexing rules.
- High-level queries about non-regulatory internal topics (e.g., "Do we have a remote work policy?").

> **Rule of Thumb:** If the answer influences a legal decision, financial agreement, compliance status, or safety protocol—load and quote the full source document.

---

## 3. ARCHITECTURAL PITFALLS & EDGE CASES

| Issue / Pitfall | Cause | Mitigation / Solution |
| :--- | :--- | :--- |
| **Tool Call Lockout** | `read_file` tools often have call limits per exact file path. | Once a file has been read, use the cached content in session memory or switch to search/grep tools rather than re-reading the same path. |
| **Table Data Loss in PDF Converts** | Automated PDF-to-Markdown tools frequently drop multi-column bounds or numerical ranges. | Treat converted tables as secondary references; mark documents containing complex tables as `verification_status: unverified`. |
| **Table of Contents (TOC) Stubs** | Standard code/regulation files may contain only index stubs without full clause text. | Verify whether the file contains full text or merely a TOC. If it's a TOC, fetch the required clause directly or consult primary documentation. |
| **YAML Regex False Positives** | Naive regex patterns (e.g., `^\w+:`) accidentally match Markdown headers instead of YAML keys. | Use targeted patterns that mandate frontmatter context and explicit key matching (e.g., `^title:\s*"([^"]*)"`). |
| **Truncated Frontmatter Parsing** | Reading only the first 200–300 bytes of a file. | Set initial scan buffers to 2,000+ characters to capture large frontmatter blocks completely. |
