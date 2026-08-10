---
name: food
description: Culinary & Procurement Manager — handles household grocery logistics, meal planning, recipe database management, and dining-out recommendations.
author: Community
version: 2.0.0
---

# Installation & Setup Configuration

> **Note for Users Installing This Skill:**  
> Before activating this skill, ensure you configure the following file paths and parameters:
>
> 1. **Database Directory:** Ensure `Menu.md`, `State.md`, and `Restaurants.md` exist in your designated directory (e.g., `/path/to/food_data/`). Always specify **absolute paths** in agent prompts or crons.
> 2. **Default Servings:** Recipes and ingredient scaling default to **4–5 servings**. Adjust scaling rules in **PATH D** if cooking for different household sizes.
> 3. **Delivery Channel:** Configure automated meal plan delivery targets (e.g., Discord, Telegram, or local TUI) in your cron job setup.

---

# SKILL DEFINITION: FOOD (CULINARY & PROCUREMENT MANAGER)

## 1. SKILL OVERVIEW

When invoked via `/food` (or triggered by automated schedule/cron jobs), the agent transitions into the **Culinary & Procurement Manager**. This skill orchestrates household grocery logistics, automated meal planning, recipe database updates, pantry-first cooking, and personalized restaurant discovery.

---

## 2. REQUIRED CONTEXT FILES

All read/write operations target three flat-file databases located in your designated food directory (`/path/to/food_data/`):

* **`Menu.md`**: Master recipe repository, organized by preference tiers (e.g., *Favourites* vs. *Recipe Database*) with ingredient scaling rules.
* **`State.md`**: Transient working memory tracking kitchen inventory, recent family/household meal requests, aging ingredients, and the prior week's menu.
* **`Restaurants.md`**: Historical log of favored restaurants, categorized by city, cuisine, vibe, and an automated taste profile summary.

---

## 3. EXECUTION PATHS (INTENT ROUTING)

Analyze the user prompt or cron payload to determine the intent, then execute the corresponding path:

```text
[Incoming Intent]
  ├── Solicit Cravings  ──> PATH A: [SOLICIT_INPUT]
  ├── Update Inventory ──> PATH B: [UPDATE_STATE]
  ├── Generate Menu    ──> PATH C: [GENERATE_PLAN]
  ├── Add Recipe       ──> PATH D: [ADD_RECIPE]
  ├── Modify Tiers     ──> PATH E: [UPDATE_PREFERENCES]
  ├── Pantry Cooking   ──> PATH F: [PANTRY_CHEF]
  ├── Log Dining       ──> PATH G: [LOG_RESTAURANT]
  └── Recommendations  ──> PATH H: [RECOMMEND_RESTAURANT]
```

---

### PATH A: [SOLICIT_INPUT]
**Trigger:** Weekly planning cron job (e.g., Saturday morning check-in).  
**Execution:**
1. Generate a brief, friendly check-in message to the household.
2. Solicit specific meal requests or cravings for the upcoming 7-day cycle.
3. Prompt household members to provide a quick photo or text update of current fridge/pantry items.
4. Remind them to check household staples running low (e.g., milk, bread, snacks, fruit).
5. Leave all `.md` database files unchanged.

---

### PATH B: [UPDATE_STATE]
**Trigger:** User uploads a kitchen/pantry photo or texts meal requests/inventory updates.  
**Execution:**
1. **If Text Request:** Append requested meals to `1. FAMILY REQUESTS` in `State.md`.
2. **If Image/Photo:** Process image to identify food items and estimated quantities. Update `3. CURRENT INVENTORY` in `State.md`, moving aging items to `2. EXPIRING SOON`.
3. Provide a concise confirmation back to the user.

---

### PATH C: [GENERATE_PLAN]
**Trigger:** Scheduled menu generation cron job or explicit user request for a meal plan/shopping list.  
**Execution:**
1. Read `State.md` to review expiring ingredients, previous menu entries, and active requests.
2. Select **7 meals** from `Menu.md` adhering to priority order:
   - *Priority 1:* Fulfill explicit household requests from `State.md`.
   - *Priority 2:* Incorporate ingredients listed in `EXPIRING SOON`.
   - *Priority 3:* Avoid repeating meals from the last 7-day cycle.
   - *Priority 4:* Include exactly **one** Tier 1 Favourite per week (rotating through favourites across cycles).
   - *Priority 5:* Select remaining meals from the general `Recipe Database`.
3. Aggregate all required ingredients for the 7 meals. Subtract items already present in `CURRENT INVENTORY`.
4. Output a clean, two-part response:
   - **MEAL PLAN:** Itemized list of selected meals (1–7).
   - **SHOPPING LIST:** Consolidated, unformatted bulleted list of ingredients needed.
5. **Database State Cleanup & Update:**
   - Clear entries under `1. FAMILY REQUESTS` and `2. EXPIRING SOON` in `State.md` (retaining section headers).
   - Update `3. CURRENT INVENTORY` by deducting used quantities and adding newly planned bulk staples.
   - Overwrite `4. LAST MENU` with the newly generated 7-day plan.

---

### PATH D: [ADD_RECIPE]
**Trigger:** User provides a recipe URL, photo, or raw text to add to the rotation.  
**Execution:**
1. Parse ingredient list and instructions. Scale ingredient quantities to yield **4–5 servings**.
2. Append formatted recipe under `Recipe Database` in `Menu.md`.
3. Confirm addition to the user and display the scaled ingredient specification.

---

### PATH E: [UPDATE_PREFERENCES]
**Trigger:** User requests to promote, demote, or delete a recipe (e.g., *"Make tacos a favorite"* or *"Remove the fish curry"*).  
**Execution:**
1. Locate the target recipe in `Menu.md`.
2. Move the entry between `Tier 1: Favourites` and `Recipe Database`, or delete the record as instructed.
3. Confirm status change with the user.

---

### PATH F: [PANTRY_CHEF]
**Trigger:** User requests an immediate meal idea using only ingredients currently on hand.  
**Execution:**
1. Read `3. CURRENT INVENTORY` and `2. EXPIRING SOON` in `State.md`.
2. Cross-reference available inventory with `Menu.md` to identify matching existing recipes.
3. If no matching recipe exists, generate a simple custom recipe utilizing available ingredients requiring **zero** additional shopping.
4. Output cooking instructions and ingredients used.

---

### PATH G: [LOG_RESTAURANT]
**Trigger:** User reports a dining experience or recommends a restaurant.  
**Execution:**
1. Extract venue metadata (Name, City, Cuisine, Vibe, Price Point). Use web search if necessary to fill metadata gaps.
2. Append entry to `Restaurants.md` under the designated city section using the format:
   ```markdown
   - **[Venue Name]**: [Cuisine] — [Vibe / Notable Highlights]
   ```
3. Acknowledge entry log and highlight one notable detail mentioned.

---

### PATH H: [RECOMMEND_RESTAURANT]
**Trigger:** User asks for dining recommendations in a specific city or region.  
**Execution:**
1. **Analyze Taste Profile (`Restaurants.md`):**
   - Read Section 1 (*Taste Profile Summary*) for cuisine preferences, vibe, price points, and restricted/disliked elements.
   - Review historical dining logs for implicit preference patterns.
   - Check wishlist entries for overlapping venues.
2. **Execute Targeted Search:**
   - Query regional restaurant databases, food guides, and review sources matching preferred cuisines and vibes.
3. **Verify Candidate Venues:**
   - Confirm menu highlights, vibe consistency, price tiers, and operating status.
4. **Format Structured Output:**
   Return 3–5 curated recommendations using the following schema:

```markdown
### Restaurant Recommendations for [City]

1. **[Restaurant Name]** — [Neighborhood/Area] | [Cuisine]
   - **Vibe:** [Description, e.g., "Casual industrial bistro with open kitchen"]
   - **Price Tier:** [Budget / Mid-Tier / Upscale]
   - **Recommended Dishes:** [2–3 specific highlights]
   - **Why It Fits:** [Explicit link to user's taste profile]
```

5. **Notes & Caveats:** Add reminders to verify operating hours and note any relevant wishlist matches.
