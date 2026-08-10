---
name: fishing-report
description: Evaluates weather, hydrological, marine, and solunar forecast data against customizable fishery criteria to produce scored fishing intelligence reports for any body of water.
version: 1.0.0
author: community
target_runtime: Hermes Agent
---

# 🎣 Universal Fishing Report Evaluator Skill

Evaluates multi-day environmental forecast data (atmospheric pressure, wind, temperature, river discharge/flow, tides, swell, and solunar cycles) against localized fishery criteria to produce structured, mobile-friendly fishing intelligence reports for any style of fishing.

---

## ⚙️ Post-Installation & Configuration Setup

Before executing this skill, ensure the following local variables and configuration files exist in your Hermes workspace:

### 1. File Paths & Config Dependencies
* `WORKSPACE_DIR`: Path to your local fishing skill configuration folder (`~/.hermes/skills/fishing-report/`).
* `LOCATIONS_FILE`: Absolute path to your location & waterbody definitions (`~/.hermes/skills/fishing-report/config/locations.md`).
* `SPECIES_FILE`: Absolute path to your target species & seasonal tactics (`~/.hermes/skills/fishing-report/config/species.md`).
* `SCORING_FILE`: Absolute path to your custom scoring matrix and environmental safety limits (`~/.hermes/skills/fishing-report/config/scoring-rules.md`).

### 2. Upstream Data Sources
* **Weather & Environmental Pipeline:** Weather script (`~/.hermes/scripts/weather.py`), USGS/Environment Agency river gauges, or Open-Meteo REST API providing relevant parameters for your fishery type:
  * **Atmospheric:** Time-bucketed wind vectors (speed, direction, gusts), barometric pressure trends, air temp, cloud cover/precipitation.
  * **Hydrological / Marine (if applicable):** River flow discharge (CFS / $m^3/s$), water level/stage, sea surface temperature (SST), swell height/period, tide peak times.
  * **Solunar & Lunar:** 28-day synodic lunar phase calculations and major/minor solunar feeding windows.

### 3. System Dependencies & Gateways
* **CLI Utilities:** Python 3 (`python3`), `curl`, `jq`.
* **Delivery Gateway:** Discord, Telegram, or iMessage webhook (uses emoji list format; avoids Markdown pipe tables for mobile layout stability).

---

## 🚀 Usage & Execution Rules

### Step 1: Filter Evaluation Windows
Filter incoming forecast JSON data to evaluate target fishing days (e.g., upcoming weekend, specific hatch dates, or custom schedule specified by prompt/cron):
* Parse date timestamps (`YYYY-MM-DD`).
* Exclude historical dates or immediate same-day sessions if evaluating upcoming trip windows.

---

### Step 2: Apply Hard Disqualifiers (Safety & Unfishable Conditions)
Read disqualification thresholds from `config/scoring-rules.md`. For every filtered day, check against hard limits specific to the targeted fishery type:
* **Freshwater / Rivers:** Excessive flood discharge, unsafe river stage, ice-lock, or severe water turbidity.
* **Coastal / Surf / Kayak:** Extreme swell height/period, hazardous wave wash, or unsafe onshore winds.
* **Lakes / Offshore:** Gale force wind gusts, severe thunderstorm/lightning probability, or extreme heat/freezing.

> **Completion criterion:** Every evaluated day is tagged as **Disqualified (RED LIGHT)** or **Cleared for Scoring**.

---

### Step 3: Opportunity Matrix Scoring (Max 10 Points)
For all cleared days, compute a total score out of 10 based on rules defined in `config/scoring-rules.md`:

1. **Atmospheric & Wind Conditions (Max 3 pts):**
   * **3 pts:** Favorable wind direction/speed for target spot; falling or stable barometric pressure trend ($1010\text{--}1018\text{ hPa}$).
   * **2 pts:** Moderate wind or neutral pressure trend.
   * **0 pts:** High sustained winds or sharp rising pressure after a cold front.

2. **Hydrological & Aquatic Metrics (Max 3 pts):**
   * **3 pts:** Optimal water flow rate (CFS / $m^3/s$), water temperature in species thermal band, or ideal wave height/period.
   * **2 pts:** Slightly low/high water levels or marginal water temperature.
   * **0 pts:** Blown-out river flows, extreme water temperature, or flat/blown-out seas.

3. **Solunar, Lunar & Tidal Alignment (Max 2 pts):**
   * **2 pts:** New Moon or Full Moon ($\pm 0.08$ phase index) / peak solunar feeding window alignment / spring tide movement.
   * **1 pt:** Moderate lunar phase or neap tide exchange.
   * **0 pts:** Quarter moons or dead water periods.

4. **Diurnal & Peak Window Timing (Max 2 pts):**
   * **2 pts:** Key activity window (e.g., high tide, hatch timing, or feeding peak) aligns with Dawn, Dusk, or low-light overcast conditions.
   * **1 pt:** Activity window occurs mid-morning or late afternoon.
   * **0 pts:** Peak window occurs at midday under bright, direct sunlight.

---

### Step 4: Classify Trip Quality
Map the aggregate score to trip classifications:
* 🟢 **GOLDEN TRIP (9–10 pts):** Exceptional alignment. Prime window for trophy target species or high catch rates.
* 🟢 **GREEN LIGHT (7–8 pts):** Solid fishable conditions. High probability of good activity.
* 🟡 **AMBER LIGHT (5–6 pts):** Marginally fishable. High effort, secondary species or technical presentation required.
* 🔴 **RED LIGHT (< 5 pts or Disqualified):** Unsafe, blown out, or poor probability.

---

### Step 5: Inject Seasonal Species & Tactical Context
Cross-reference current date/season with `config/species.md`:
* Identify primary and secondary target species active in the current seasonal window.
* Append recommended flies, lures, baits, rigs, line weights, and habitat focus areas (e.g., deep pools, weed lines, structure, or gutters).

---

### Step 6: Generate Intelligence Report
Format output for mobile messaging platforms (Discord/Telegram/iMessage). **Do not use Markdown pipe tables.**

#### Report Structure:
1. 🚨 **Executive Summary & Best Window Flag**
2. 🏖️ **Primary Forecast Days (Emoji Bullet Format)**
3. 📅 **Extended Outlook Preview**
4. 🌕 **Lunar, Solunar & Aquatic Condition Summary**
5. 🚜 **Access, Logistics & Safety Advisory**

---

## 📋 Standard Output Format Example

```text
🚨 EXECUTIVE SUMMARY
Target Waterbody: Pine River (Lower Reach)
Best Window: Saturday Morning (Score: 9/10 - GOLDEN TRIP)
Primary Targets: Brown Trout, Rainbow Trout

🟢 Sat Aug 15 (GOLDEN TRIP) ⭐ Top Pick
• Wind: 5kn SW (Light) — Dawn window
• Flow / Temp: 180 CFS (Optimal) | Water Temp: 12°C
• Atmospheric: Barometer 1012 hPa (Falling) — Overcast
• Solunar: Full Moon (0.98) — Major feeding window 06:15 - 08:15
• Score: 9/10 — Perfect flow rate + falling barometer + dawn hatch

🚫 Sun Aug 16 (RED LIGHT)
• Flow / Weather: 850 CFS ❌ — Disqualified: Exceeds max safe discharge (400 CFS)
• Wind: NW 28kt — Heavy rain & flash flood warning

🚜 ACCESS & LOGISTICS
• Wading Safety: Caution advised near gorge entrance due to recent rainfall.
• Recommended Gear: 5wt fly rod, 4X tippet, size 16 Mayfly nymphs / streamers.
```
