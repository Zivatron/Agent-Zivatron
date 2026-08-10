---
name: cron-manager
description: Write, edit, and debug cron jobs for Hermes Agent. Use when creating new cron jobs, modifying existing ones, or troubleshooting cron failures.
author: Community
version: 1.0.0
---

# Installation & Setup Configuration

> **Note for Users Installing This Skill:**  
> Before activating or running cron jobs managed by this skill, ensure you configure the following environment-specific placeholders in your job prompts:
>
> 1. **File Paths:** Always replace `/Users/YOUR_USERNAME/...` or `/home/YOUR_USERNAME/...` with full, absolute paths native to your system.
> 2. **Profile Directory:** Standard Hermes installations keep cron jobs at `~/.hermes/cron/jobs.json`. If using multi-profile setups, update paths to `~/.hermes/profiles/<PROFILE_NAME>/cron/jobs.json`.
> 3. **Delivery Targets:** Replace platform placeholders (`discord:CHANNEL_ID`, `telegram:CHAT_ID`) with your real channel IDs or chat IDs.

---

# SKILL DEFINITION: CRON MANAGER

## 1. THE GOLDEN RULE — READ BEFORE YOU EDIT

**Never edit a cron job from the `cronjob(action='list')` preview.** The prompt is truncated — you will miss critical context and break things.

*Note on Profiles:* If using a named Hermes profile, the file path will be `~/.hermes/profiles/<PROFILE_NAME>/cron/jobs.json`.

**Always read the full job from disk:**

```bash
cat ~/.hermes/cron/jobs.json | python3 -c "import json,sys; data=json.load(sys.stdin); jobs=data.get('jobs',[]); job=[j for j in jobs if 'JOB_NAME_KEYWORD'.lower() in str(j.get('name','')).lower()]; print(json.dumps(job, indent=2))"
```

Replace `JOB_NAME_KEYWORD` with a unique substring from the job name. If multiple jobs match, filter by `job_id` instead:

```bash
cat ~/.hermes/cron/jobs.json | python3 -c "import json,sys; data=json.load(sys.stdin); jobs=data.get('jobs',[]); job=[j for j in jobs if j.get('id')=='JOB_ID']; print(json.dumps(job, indent=2))"
```

---

## 2. CORE PRINCIPLES

### Zero-Context Execution
Cron jobs start in a fresh, isolated sub-session. They have no conversation history, no session memory, and cannot ask questions. Every piece of context the agent needs must be in the prompt or loaded via skills/files.

### 5-Minute Local LLM Buffer
Local LLM instances cannot process parallel background requests efficiently. Every scheduled job MUST be offset by at least 5 minutes from any existing cron job.

### Jitter Staggering
Never schedule recurring jobs on top-of-the-hour marks (e.g., `0 8 * * *`). Offset them to prevent cron collisions and LLM resource contention (e.g., `7 8 * * *` or `12 9 * * *`).

### Absolute Paths
Cron jobs run in isolated working directories. Always use absolute paths (`/Users/YOUR_USERNAME/workspace/documents/...`) not relative ones (`./data.txt`).

### Explicit Platform Routing
Always explicitly specify where results should go — don't rely on defaults. Specify the exact Discord channel, Telegram chat ID, or delivery target.

---

## 3. CRON JOB ARCHITECTURE

A cron job has three layers:

### Layer 1 — The Prompt (task instruction)
The self-contained instructions the agent follows each tick. Must be fully self-contained — cron sessions have no conversation history and cannot ask questions.

### Layer 2 — The Skill (procedural knowledge)
Skills loaded via the `skills` field provide reusable procedures. The prompt delegates to them rather than duplicating logic.

### Layer 3 — Delivery (where output goes)
`deliver` controls where the result is sent:
- `origin` — back to the chat/topic that created it (default)
- `discord:CHANNEL_ID` — specific Discord channel
- `telegram:CHAT_ID:THREAD_ID` — specific Telegram thread
- `all` — fan out to every connected channel
- `local` — save only, no delivery

---

## 4. WRITING CRON PROMPTS

### Structure
```text
You are running the [JOB_NAME] cron job. Follow these steps exactly:
1. Load the `SKILL_NAME` skill (skill_view).
2. Read [relevant files with absolute paths].
3. Execute the relevant PATH from the skill.
4. Output the final result to the user in this chat.
```

### Pre-flight Checklist
Before writing or editing any cron job, verify:
- [ ] **Goal:** What exact task to perform is declared.
- [ ] **Target Paths:** Explicit absolute paths (e.g., `/Users/YOUR_USERNAME/workspace/documents/...`).
- [ ] **Tools Allowed:** Explicit instructions on which tools to call.
- [ ] **Delivery Channel:** Where results should go (not relying on defaults).
- [ ] **Fallback/Error Behavior:** What to do if an external call fails.

### Principles
- **Delegate to skills.** The cron prompt should be a thin orchestrator — load the skill, read files, execute the relevant PATH. Don't duplicate skill logic in the prompt. If you find yourself writing "but with this important modification to step X" inside a cron prompt, that's a signal the skill needs updating instead.
- **One job, one purpose.** Each cron should do one thing well. If a prompt is doing three different things, split it into separate jobs.
- **Explicit completion.** End with a clear deliverable: *"Output the final summary to the user in this chat."*

### Anti-Patterns

| Anti-Pattern | Why It Breaks | Correct Pattern |
| :--- | :--- | :--- |
| Editing based on `cronjob(action='list')` output | `list` truncates prompt text; editing overwrites the job with incomplete text | Read `~/.hermes/cron/jobs.json` first to get full text |
| Scheduling multiple jobs at `00` minutes (e.g., `0 8 * * *`) | Local LLM inference conflicts, leading to timeouts and dropped triggers | Stagger by off-peak minutes (e.g., `7 8 * * *`, `14 9 * * *`) |
| Scheduling jobs within 5 minutes of each other | Local GPU/CPU queues lock up | Maintain at least **5-minute gap** between jobs |
| Using relative paths in prompts (`./data.txt`) | Cron working directory may differ from user chat context | Use absolute paths (`/Users/YOUR_USERNAME/data.txt`) |
| Assuming chat context exists ("as discussed earlier") | Cron sessions run in isolated sub-agents without chat memory | Put all instructions, URLs, and rules inside the prompt |
| Prompt overrides skill | Modifying a PATH's output in the cron instead of fixing it in the skill | Update the skill; keep the prompt as a thin delegator |

---

## 5. EDITING CRON JOBS

Use `cronjob(action='update')` with the full replacement prompt. The entire prompt is replaced — not patched.

**Before updating:**
1. Read the full job from `~/.hermes/cron/jobs.json` (Golden Rule).
2. Check the 5-minute buffer against all other active jobs — adjust schedule if needed.
3. Identify what needs to change and why.
4. Draft the new prompt preserving everything that still works.

**After updating:**
1. Verify with another read from disk — confirm the change applied correctly.
2. Check `last_status` and `next_run_at` are sane.

---

## 6. DEBUGGING CRON FAILURES

When a cron job fails:
1. **Read the full prompt** from disk (Golden Rule).
2. **Check `last_error`** in the job JSON for the error message.
3. **Verify skills still exist** — `skills_list` to confirm referenced skills are available.
4. **Check file paths** — do the files the cron reads still exist at those absolute locations?
5. **Check schedule collisions** — is another job firing within 5 minutes and blocking the local LLM?
6. **Run manually** with `cronjob(action='run', job_id='...')` to test the fix.
7. **Check delivery** — if the job succeeds but the user never sees it, verify `deliver` targets a gateway-connected platform (not `origin` when running from TUI).

---

## 7. SCHEDULE FORMATS

- **Cron expression:** `"0 9 * * 0"` — Sunday at 9am
- **Interval:** `"30m"`, `"every 2h"` — every 30 minutes, every 2 hours
- **ISO timestamp:** `"2026-08-15T09:00:00"` — one-shot at a specific time

**Common patterns (with jitter):**
- `7 9 * * 0` — Sunday 9:07am (weekly task)
- `12 10 * * 6` — Saturday 10:12am (weekend summary)
- `30 8 * * 1` — Monday 8:30am (weekly status check)
- `22 17 * * 4` — Thursday 5:22pm (recurring pipeline)

---

## 8. CRON JOB METADATA REFERENCE

Key fields in the job JSON:
- `id` — unique identifier (use with `cronjob(action='update', job_id=...)`)
- `name` — human-readable name
- `prompt` — full instruction text (always read from disk)
- `skills` — list of skill names loaded before execution
- `schedule` / `schedule_display` — cron expression and human-readable form
- `deliver` — output destination (e.g., `discord:CHANNEL_ID`)
- `enabled` / `state` — active or paused
- `last_run_at` / `next_run_at` — timing info
- `last_status` — "ok" or "error"
- `last_error` — error message if failed
- `model_snapshot` / `provider_snapshot` — pinned model at creation time

---

## 9. TOOL ACTIONS REFERENCE

| Action | Purpose | Required Fields |
| :--- | :--- | :--- |
| `create` | Schedule a new job | `schedule`, `prompt` |
| `list` | List all jobs (truncated prompts — DO NOT use for editing) | none |
| `update` | Modify an existing job's prompt/schedule/deliver | `job_id`, plus any fields to change |
| `pause` | Temporarily stop a job | `job_id` |
| `resume` | Restart a paused job | `job_id` |
| `remove` | Delete a job permanently | `job_id` |
| `run` | Execute a job immediately for testing | `job_id` |
