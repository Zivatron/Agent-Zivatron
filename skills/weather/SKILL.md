---
name: weather
description: Fetch current, forecasted weather, wind, and coastal marine conditions via Open-Meteo APIs.
version: 1.0.0
author: community
target_runtime: Hermes Agent
---

# 🌤️ Weather & Marine Conditions Skill

A zero-authentication weather and coastal ocean forecast skill powered by Open-Meteo APIs. Resolves geographic location names to coordinates, queries 14-day weather forecasts and 10-day marine/swell models, and synthesizes reports tailored to general queries or surf/coastal activities.

---

## ⚙️ Post-Installation & Configuration Setup

Before executing this skill, ensure the following local variables, helper scripts, and paths are set up in your Hermes environment:

### 1. File Paths & Environment Variables
* `SCRIPTS_DIR`: Path to local Hermes executable scripts (e.g., `~/.hermes/scripts/`).
* `WEATHER_SCRIPT`: Path to the Python weather fetcher script (`~/.hermes/scripts/weather.py`).
* `REFERENCES_DIR`: (Optional) Path to regional coordinate lookup tables (e.g., `~/.hermes/skills/weather/references/coords.md`).

### 2. Local Runtimes & Network
* **Outbound Traffic:** Requires unrestricted outbound HTTP/HTTPS access to `geocoding-api.open-meteo.com` and `api.open-meteo.com`.
* **Ports:** None required (stateless REST client).

### 3. System Dependencies & Gateways
* **Python 3:** System Python runtime (`python3`) installed with standard libraries (`urllib`, `json`, `argparse`, `math`).
* **CLI Utilities:** `curl` available in the system PATH.

---

## 🚀 Usage & Execution Rules

Trigger this skill whenever the user asks about weather, temperature, wind, swell/wave forecasts, or coastal water conditions.

### Step 1: Location Resolution
Resolve the location query to latitude and longitude coordinates:
1. **Local Lookup Table:** Check `references/coords.md` if available for pre-defined regional locations.
2. **Geocoding API Fallback:** If not listed locally, query the Open-Meteo Geocoding endpoint:
   ```bash
   curl -s "[https://geocoding-api.open-meteo.com/v1/search?name=](https://geocoding-api.open-meteo.com/v1/search?name=)<LOCATION_NAME>&count=1"
   ```
   *Extract the numeric `latitude` and `longitude` fields from the first result.*

### Step 2: Script Execution
Execute the weather calculation script with resolved coordinates:
```bash
python3 ~/.hermes/scripts/weather.py <LATITUDE> <LONGITUDE> --label "<DISPLAY_NAME>"
```

* **Coastal/Swell Mode:** Append `--ocean-only` if the query focuses strictly on waves, surf, or sea state.
* **Programmatic Mode:** Append `--json` if parsing field data downstream in an automated workflow or cron job.

### Step 3: Synthesis & Reporting
Summarize the JSON or text output returned by the script for the user:
* Highlight **current conditions**, daily temperature ranges, and notable precipitation patterns.
* For coastal/marine queries, highlight **swell height/period**, wind speed/direction shifts, and **tide implications** (derived from sea surface height).
* Explicitly state the relevant local timezone and date in the response.

---

## 📋 Examples & Templates

### Command Line Invocation
```bash
# Standard location weather
python3 ~/.hermes/scripts/weather.py -34.9285 138.6007 --label "Adelaide"

# Coastal ocean/surf query
python3 ~/.hermes/scripts/weather.py -35.2133 138.4611 --label "Sellicks Beach" --ocean-only
```

### Reference Table Format (`references/coords.md`)
```markdown
| Location Name | Latitude | Longitude | Type |
| :--- | :--- | :--- | :--- |
| Bondi Beach | -33.8915 | 151.2767 | Coastal |
| Bells Beach | -38.3683 | 144.2800 | Coastal |
| Mount Buffalo | -36.7214 | 146.7753 | Alpine |
```
