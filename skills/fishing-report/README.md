# 🎣 Universal Fishing Report Skill — Setup & Research Guide

This repository contains a modular, configuration-driven fishing evaluation skill for **Hermes Agent**. It transforms raw weather, hydrological, marine, and solunar forecast data into actionable, scored fishing intelligence reports tailored to **any fishery type** globally (freshwater rivers, trout streams, bass lakes, estuaries, deep-sea charters, kayak routes, or coastal surf beaches).

---

## 📂 File Structure

```text
skills/fishing-report/
├── README.md                  # Human setup guide & research prompts
├── SKILL.md                   # Hermes Agent execution runtime
└── config/
    ├── locations.md           # Your target spots, coordinates & waterbody specs
    ├── species.md             # Local target species, seasonal hatches & tactics
    └── scoring-rules.md       # Safety limits & scoring matrix for your fishery
```

---

## ⚙️ Installation Instructions

### Step 1: Copy Configuration Templates
Create your local configuration directory inside your Hermes skills workspace:
```bash
mkdir -p ~/.hermes/skills/fishing-report/config
```

### Step 2: Generate Your Local Configuration Files
This skill requires localized rules regarding your target body of water, local target species, and environmental/safety limits.

Use the **Universal Deep Research Meta-Prompt** in the section below to automatically generate your tailored `locations.md`, `species.md`, and `scoring-rules.md` files.

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

If you prefer to edit files manually without using the meta-prompt, use these starter formats:

### `config/locations.md`
```markdown
# Target Fishing Locations

| Location Name | Spot Lat/Lon | Waterbody / Gauge ID | Access / Logistics Notes |
| :--- | :--- | :--- | :--- |
| Pine River Gorge | 40.1234, -105.5678 | USGS Gauge #06714000 | Hike-in access. Wading staff recommended above 200 CFS. |
| Blue Heron Lake | 34.5678, -84.1234 | Reservoir Level Grid B | Public boat ramp open year-round. Electric motor only. |
```

### `config/species.md`
```markdown
# Seasonal Species Profiles

## Spring / Summer
* **Primary Target:** Brown Trout (*Salmo trutta*)
* **Tactics:** Late afternoon Mayfly hatches (Size 14-16 Parachute Adams). Target bubble lines and undercut banks.

## Autumn / Winter
* **Primary Target:** Rainbow Trout (*Oncorhynchus mykiss*)
* **Tactics:** Deep nymphing with indicator rigs or slow-stripped streamer flies (Woolly Buggers) in deep pools.
```

### `config/scoring-rules.md`
```markdown
# Environmental & Safety Thresholds

## Hard Disqualifiers
* **Max Flow Rate / Discharge:** > 400 CFS (River too high/dangerous to wade)
* **Max Sustained Wind:** > 25 knots
* **Severe Weather:** Thunderstorms / Lightning probability > 40%

## Scoring Matrix (10 Points Total)
* **Atmospheric (3 pts):** Barometer falling (1008-1014 hPa) + Wind < 10kn = 3 pts | Neutral = 2 pts | Rising post-front = 0 pts
* **Hydrological (3 pts):** Flow 150-250 CFS = 3 pts | 100-149 or 251-350 CFS = 2 pts | Blown out / Trickle = 0 pts
* **Solunar & Lunar (2 pts):** New/Full Moon or Major Feeding Window = 2 pts | Minor Window = 1 pt | Off-peak = 0 pts
* **Timing (2 pts):** Peak window aligns with Dawn/Dusk = 2 pts | Overcast daytime = 1 pt | Bright mid-day sun = 0 pts
```

---

## ✨ Key Features & Capabilities

1. **Universal Waterbody Support:** Operates seamlessly across rivers, lakes, reservoirs, estuaries, offshore deep sea, and surf beaches.
2. **Flexible Environmental Modeling:** Handles barometric pressure trends, river discharge/flow rates (CFS or $m^3/s$), water temperature, solunar feeding peaks, tides, wind, and ocean swell.
3. **Automated Setup via Research Prompt:** Bootstraps complete configuration files for any global fishing spot using a tailored AI deep research prompt.
