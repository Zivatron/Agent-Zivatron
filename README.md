# Agent-Zivatron
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
| **[`lmstudio-vision-fix`](skills/lmstudio-vision-fix)** | Fixes HTTP 400 (`Invalid messages`) payload bugs. | Configures `agent.image_input_mode: native` for local vision models (Qwen, MLX) in LM Studio. | Universal |

---

## 📋 Starter Templates

For state-dependent skills, example databases are packaged alongside the skill file:

* **[`skills/food/State.example.md`](skills/food/State.example.md)** — Starter flat-file state for tracking active household requests, expiring inventory, and past menu history.
* **[`skills/food/Menu.example.md`](skills/food/Menu.example.md)** — Starter recipe database structure categorized by Tier 1 Favourites and general recipe items.

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
    ├── food/
    │   ├── SKILL.md
    │   ├── Menu.example.md    # Starter recipe database
    │   └── State.example.md   # Starter kitchen inventory state
    ├── librarian/
    │   └── SKILL.md
    └── lmstudio-vision-fix/
        └── SKILL.md
```

---

## 🚀 Installation & Usage

### Option 1: Browser / Direct Download (Single Skill)
To install a single skill into your local Hermes installation:

1. Create the skill directory in your local Hermes configuration:
   ```bash
   mkdir -p ~/.hermes/skills/food
   ```
2. Download the raw `SKILL.md` directly into that folder:
   ```bash
   curl -o ~/.hermes/skills/food/SKILL.md \
     [https://raw.githubusercontent.com/Zivatron/Agent-Zivatron/main/skills/food/SKILL.md](https://raw.githubusercontent.com/Zivatron/Agent-Zivatron/main/skills/food/SKILL.md)
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

When using skills that rely on local storage or external tools:

1. **Check Environment Placeholders:** Search installed `SKILL.md` files for placeholders like `YOUR_USERNAME` or `/path/to/data/` and replace them with your local absolute paths.
2. **Setup State Files:** For state-dependent skills like `food`, copy `skills/food/State.example.md` and `skills/food/Menu.example.md` into your local food document workspace as `State.md` and `Menu.md`.
3. **Restart Hermes Gateway:**
   ```bash
   hermes gateway restart
   ```

---

## 📄 License

Distributed under the **MIT License**. Free for personal, commercial, and community adaptation.
