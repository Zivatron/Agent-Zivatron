# 🎣 Universal Fishing Report Skill — Setup & Research Guide

This repository contains a modular, configuration-driven fishing evaluation skill for **Hermes Agent**. It transforms raw weather, hydrological, marine, and solunar forecast data into actionable, scored fishing intelligence reports tailored to **any fishery type** globally (freshwater rivers, trout streams, bass lakes, estuaries, deep-sea charters, kayak routes, or coastal surf beaches).

---

## 📂 File Structure

```text
skills/fishing-report/
├── README.md                          # Human setup guide & research prompts
├── SKILL.md                           # Hermes Agent execution runtime
└── config/
    ├── locations.example.md           # Starter locations template
    ├── species.example.md             # Starter species template
    └── scoring-rules.example.md       # Starter safety & scoring rules template
```

---

## ⚙️ Installation Instructions

### Step 1: Create Local Config & Copy Templates
Create your local configuration directory inside your Hermes skills workspace and copy the starter template files into active `.md` config files:

```bash
# Create local configuration directory
mkdir -p ~/.hermes/skills/fishing-report/config

# Copy starter templates to active config files
cp skills/fishing-report/config/locations.example.md ~/.hermes/skills/fishing-report/config/locations.md
cp skills/fishing-report/config/species.example.md ~/.hermes/skills/fishing-report/config/species.md
cp skills/fishing-report/config/scoring-rules.example.md ~/.hermes/skills/fishing-report/config/scoring-rules.md
```

### Step 2: Populate Your Active Configuration Files
Edit the newly created `locations.md`, `species.md`, and `scoring-rules.md` files with your local fishery details.

You can either edit them manually or use the **Universal Deep Research Meta-Prompt** in the section below to automatically generate customized content for your specific fishing spots.

---

## 🧪 Universal Deep Research Meta-Prompt

Copy and paste the prompt below into an LLM (Gemini, ChatGPT, or Claude) along with details about your **fishery type and target body of water** to generate your localized configuration files.

```text
I am setting up an automated fishing evaluation skill for my local fishery.
Please conduct a deep research assessment for the following fishery:
- Target Waterbody / Spot Name: [INSERT WATERBODY NAME / LOCATION]
- Fishery Type: [e.g., Trout Fly Fishing / Bass Lake / Estuary Lure / Deep Sea Offshore / Surf Beach]
- Region / Country: [INSERT REGION, STATE, COUNTRY]

Generate three distinct Markdown files formatted exactly as requested below:

---

### FILE 1: `locations.md`
Provide exact geographic coordinates and waterbody specs:
1. Primary spot name, latitude, and longitude (decimal format).
2. Waterbody specs (e.g., river discharge gauge ID, lake elevation, offshore sector name, or marine grid).
3. Access notes (parking, boat ramp status, wading conditions, 4WD requirements, or kayak launch spots).
4. Local regulations or closures (e.g., seasonal catch-and-release zones, permit requirements, sanctuary areas).

---

### FILE 2: `species.md`
Categorize target species by season for this location:
1. **Primary Season 1 (e.g., Spring/Summer):** Main target species, optimal water temperature range, active feeding windows (e.g., insect hatches, bait migrations, tide phase), recommended lures/flies/baits, line class, and habitat structure.
2. **Primary Season 2 (e.g., Autumn/Winter):** Main target species, tactics, lure/bait presentations, and key depth zones.

---

### FILE 3: `scoring-rules.md`
Define numeric environmental safety limits and optimal conditions for this specific style of fishing:
1. **Hard Disqualifiers (RED LIGHT / Unfishable):**
   - Maximum/minimum safe water flow discharge (CFS or m³/s) OR maximum safe swell height (m).
   - Maximum sustained wind speed or gust threshold (knots or mph).
   - Severe weather triggers (e.g., lightning probability, dangerous heat index, or freezing air temp).
2. **Scoring Categories (Max 10 Points):**
   - Atmospheric & Wind (Barometric pressure trends, wind direction/speed relative to spot).
   - Hydrological / Aquatic Metrics (Ideal flow rate, water temp, clarity, or wave metrics).
   - Solunar & Tidal / Lunar Alignment (Feeding windows, tide exchange, moon phase).
   - Diurnal Timing (High activity alignment with Dawn, Dusk, or hatch times).
```

---

## 📋 Starter Configuration Templates

If you prefer to edit files manually without using the meta-prompt, reference the starter files in `config/*.example.md`:

* **[`config/locations.example.md`](config/locations.example.md)** — Starter template for target coordinates, gauge IDs, and access notes.
* **[`config/species.example.md`](config/species.example.md)** — Starter template for seasonal species profiles, thermal zones, and tactics.
* **[`config/scoring-rules.example.md`](config/scoring-rules.example.md)** — Starter template for environmental safety thresholds and the 10-point scoring matrix.

---

## ✨ Key Features & Capabilities

1. **Universal Waterbody Support:** Operates seamlessly across rivers, lakes, reservoirs, estuaries, offshore deep sea, and surf beaches.
2. **Flexible Environmental Modeling:** Handles barometric pressure trends, river discharge/flow rates (CFS or $m^3/s$), water temperature, solunar feeding peaks, tides, wind, and ocean swell.
3. **Automated Setup via Research Prompt:** Bootstraps complete configuration files for any global fishing spot using a tailored AI deep research prompt.
