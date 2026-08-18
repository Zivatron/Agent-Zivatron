# 🤖 Hermes Agent Skills & Templates

A curated, community-driven collection of production-ready skills, state management templates, and integration workflows designed specifically for [Hermes Agent](https://github.com/nousresearch/hermes-agent) and local LLM environments.

---

## 📌 Rationale & Target Runtime

These skills were engineered specifically for **Hermes Agent** running on local hardware (macOS / Linux) with local inference backends (LM Studio, MLX, vLLM, Ollama) and gateway integrations (Discord, Telegram, iMessage via BlueBubbles).

They address real-world local agent constraints, such as:
* **Zero-context sub-sessions:** Ensuring cron jobs and automated tasks carry all required context without depending on chat memory.
* **Local LLM resource contention:** Enforcing 5-minute jitter offsets to prevent parallel GPU/CPU locking.
* **Payload compatibility:** Workarounds for local OpenAI-compatible endpoints handling multimodal/vision tool calls.
* **Strict RAG citation & safety:** Mandatory disclaimers for PDF-to-Markdown document conversions.

---

## 📦 Skill Catalog

| Skill Name | Description | Key Features | Target Platform |
| :--- | :--- | :--- | :--- |
| **[`cron-manager`](skills/cron-manager)** | Write, edit, and debug scheduled jobs safely. | Enforces Golden Rule (read full JSON from disk), 5-min jitter offsets, absolute path routing. | Universal |
| **[`bluebubbles`](skills/bluebubbles)** | iMessage gateway integration & health monitoring. | Self-healing webhook registration, port conflict detection, IPv4 binding (`127.0.0.1`). | macOS |
| **[`librarian`](skills/librarian)** | Document library architecture & RAG indexing. | 2,000+ char YAML frontmatter scanning, `INDEX.md` generation, strict regulatory RAG citations. | Universal |
| **[`food`](skills/food)** | Household culinary & grocery procurement manager. | 7-day meal planning, VLM/image-based pantry inventory updates, 4–5 serving recipe scaling, restaurant taste profile matching. Includes starter templates. | Universal |
| **[`weather`](skills/weather)** | Fetch weather & marine ocean conditions via Open-Meteo. | Zero-auth REST API queries, geocoding resolution, 14-day weather & 10-day swell/wave model reports. | Universal |
| **[`fishing-report`](skills/fishing-report)** | Universal multi-waterbody fishing intelligence report generator. | 10-point environmental scoring matrix, hard safety disqualifiers, AI deep research setup prompt, mobile-friendly Discord/Telegram formatting. | Universal |
| **[`lmstudio-vision-fix`](skills/lmstudio-vision-fix)** | Fixes HTTP 400 (`Invalid messages`) payload bugs. | Configures `agent.image_input_mode: native` for local vision models (Qwen, MLX) in LM Studio. | Universal |
| **[`deep-research`]([skills/deep-research])** | An autonomous research pipeline. | Credit & Budget Discipline, Anti-Pattern & Scraping Protections, Tiered Search Waterfall, Mandatory Fact Cross-Verification, Domain Authority Escalation and Flexible Structured Synthesis. | Universal |

---

## 📋 Starter Templates

For state-dependent and configuration-driven skills, example databases and config files are packaged alongside the skill folder:

* **[`skills/food/State.example.md`](skills/food/State.example.md)** — Starter flat-file state for tracking active household requests, expiring inventory, and past menu history.
* **[`skills/food/Menu.example.md`](skills/food/Menu.example.md)** — Starter recipe database structure categorized by Tier 1 Favourites and general recipe items.
* **[`skills/fishing-report/config/*.example.md`](skills/fishing-report/config)** — Starter configuration templates for target coordinates (`locations.example.md`), target species & tactics (`species.example.md`), and safety/scoring thresholds (`scoring-rules.example.md`).

---

## 📂 Repository Structure

```text
Agent-Zivatron/
├── LICENSE                    # MIT Open-Source License
├── README.md                  # Main repository documentation & skill index
└── skills/                    # Self-contained skill catalog
    ├── bluebubbles/
    │   └── SKILL.md
    ├── cron-manager/
    │   └── SKILL.md
    ├── fishing-report/
    │   ├── README.md          # Setup guide & deep research prompt
    │   ├── SKILL.md           # Hermes Agent execution runtime
    │   └── config/
    │       ├── locations.example.md
    │       ├── scoring-rules.example.md
    │       └── species.example.md
    ├── food/
    │   ├── SKILL.md
    │   ├── Menu.example.md    # Starter recipe database
    │   └── State.example.md   # Starter kitchen inventory state
    ├── librarian/
    │   └── SKILL.md
    ├── lmstudio-vision-fix/
    │   └── SKILL.md
    └── weather/
    |   └── SKILL.md
    └── deep-research/
    │   └── SKILL.md                                  # Core agent instructions & pipeline definition
        └── references/
            ├── ddgs-python-library-workflow.md       # Fast parallel query execution patterns
            └── curated-research-report-format.md     # Output templates for industry & policy briefings
```

---

## 🚀 Installation & Usage

### Option 1: Browser / Direct Download (Single Skill)
To install a single skill into your local Hermes installation:

1. Create the skill directory in your local Hermes configuration:
   ```bash
   mkdir -p ~/.hermes/skills/fishing-report
   ```
2. Download the raw `SKILL.md` directly into that folder:
   ```bash
   curl -o ~/.hermes/skills/fishing-report/SKILL.md \
     [https://raw.githubusercontent.com/Zivatron/Agent-Zivatron/main/skills/fishing-report/SKILL.md](https://raw.githubusercontent.com/Zivatron/Agent-Zivatron/main/skills/fishing-report/SKILL.md)
   ```

---

### Option 2: Clone & Symlink (Full Catalog & Auto-Updates)
To install all skills and receive updates when new skills are pushed:

1. Clone this repository locally:
   ```bash
   git clone [https://github.com/Zivatron/Agent-Zivatron.git](https://github.com/Zivatron/Agent-Zivatron.git) ~/Agent-Zivatron
   ```
2. Link the repository skill folders directly to your local `.hermes` runtime directory:
   ```bash
   # Create skills directory if it doesn't exist
   mkdir -p ~/.hermes/skills

   # Create symbolic links for all skills
   ln -s ~/Agent-Zivatron/skills/* ~/.hermes/skills/
   ```

*Now, any time you run `git pull` inside `~/Agent-Zivatron`, your local Hermes agent automatically gets the latest skill updates.*

---

## ⚙️ Post-Installation Configuration

When using skills that rely on local storage, external tools, or custom configurations:

1. **Check Environment Placeholders:** Search installed `SKILL.md` files for placeholders like `YOUR_USERNAME` or `/path/to/data/` and replace them with your local absolute paths.
2. **Setup State & Config Files:**
   * **For `food` skill:** Copy `skills/food/State.example.md` and `skills/food/Menu.example.md` into your local food workspace as `State.md` and `Menu.md`.
   * **For `fishing-report` skill:** Copy the `.example.md` files from `skills/fishing-report/config/` into `~/.hermes/skills/fishing-report/config/` as active `.md` files (`locations.md`, `species.md`, `scoring-rules.md`).
3. **Restart Hermes Gateway:**
   ```bash
   hermes gateway restart
   ```

---

## 📄 License

Distributed under the **MIT License**. Free for personal, commercial, and community adaptation.
