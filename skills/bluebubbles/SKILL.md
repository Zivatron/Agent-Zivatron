---
name: bluebubbles
description: Use when setting up, configuring, or troubleshooting BlueBubbles iMessage integration with Hermes Agent gateway — server config, webhook setup, port conflicts, self-healing diagnostics, and inbound/outbound messaging.
author: Community
version: 2.1.0
platforms:
  - macos
---

# Installation & Setup Configuration

> **Note for Users Installing This Skill:**  
> Before enabling or testing the BlueBubbles iMessage integration, ensure you configure the following settings:
>
> 1. **Alphanumeric Password:** Set an alphanumeric-only password in BlueBubbles settings (avoid special characters like `#`, `!`, or `@`).
> 2. **Port Configuration:** Ensure `BLUEBUBBLES_SERVER_URL` (e.g., port `8645`) and `BLUEBUBBLES_WEBHOOK_PORT` (e.g., port `8646`) do NOT share the same port.
> 3. **IP Address Binding:** Always use `127.0.0.1` instead of `localhost` across environment variables to prevent IPv6 (`::1`) binding mismatches on macOS.
> 4. **Phone Numbers & Addresses:** Update destination phone numbers in send requests to match international formatting (e.g., `+15551234567` or `+61400000000`).

---

# SKILL DEFINITION: BLUEBUBBLES INTEGRATION

Bridge iMessage to Hermes via the BlueBubbles macOS server. Supports text, media (images, voice notes, documents), tapback reactions, read receipts, and real-time inbound webhooks.

## 1. QUICK DECISION TREE

| Task | Section |
| :--- | :--- |
| Initial setup (install, configure, connect) | [Setup](#2-setup) |
| Inbound messages not arriving | [Self-Healing Diagnostic Routine](#4-self-healing-diagnostic-routine) |
| Outbound messages failing or timing out | [Troubleshooting: Send Failures](#5-troubleshooting-send-failures) |
| Health monitoring | [Health Checks](#6-health-checks) |

---

## 2. SETUP

### Step 1 — Install BlueBubbles on macOS
1. Download from the official BlueBubbles website.
2. Install the macOS app and sign in with your Apple ID (iMessage account).
3. Enable **Private API** and ensure **Helper Connected** in **Settings → Advanced**.

### Step 2 — Configure BlueBubbles Server
1. Open BlueBubbles → **Settings → Server**.
2. Turn on **"Enable Server"**.
3. Set port (e.g., `8645`) — must be free; verify with `lsof -i :<port>`.
4. Set API password — **MUST be alphanumeric only** (letters and numbers, no special characters).

### Step 3 — Configure Hermes (`.env`)
```bash
# Server URL — ALWAYS use 127.0.0.1, NOT localhost (macOS resolves localhost to IPv6 ::1)
BLUEBUBBLES_SERVER_URL=http://127.0.0.1:8645

# Server password — alphanumeric ONLY (no #, !, @ or other special chars)
BLUEBUBBLES_PASSWORD=<ALPHANUMERIC_PASSWORD>

# Webhook host — MUST be 127.0.0.1 (IPv4), NOT localhost
BLUEBUBBLES_WEBHOOK_HOST=127.0.0.1

# CRITICAL: Webhook port MUST differ from server port
BLUEBUBBLES_WEBHOOK_PORT=8646

# Webhook path — must be /bluebubbles-webhook
BLUEBUBBLES_WEBHOOK_PATH=/bluebubbles-webhook

# Allow all users to interact with the bot
BLUEBUBBLES_ALLOW_ALL_USERS=true
```

### Step 4 — Restart Gateway
```bash
hermes gateway restart
```

### Step 5 — Verify Connection
```bash
# Check logs:
tail -30 ~/.hermes/logs/gateway.log | grep bluebubbles
# Expected: "✓ bluebubbles connected" + webhook registered

# Test server ping (no URL encoding needed with alphanumeric password):
curl -s "http://127.0.0.1:8645/api/v1/ping?password=<BLUEBUBBLES_PASSWORD>"
# Expected: {"status":200,"message":"Ping received!","data":"pong"}

# Verify ports are separate:
lsof -i :8645   # Should show only BlueBubbles
lsof -i :8646   # Should show python3.1 (gateway)
```

---

## 3. HOW IT WORKS

| Direction | Mechanism | Endpoint |
| :--- | :--- | :--- |
| **Outbound** (Hermes → iMessage) | REST API `POST` | `/api/v1/message/text` or `/api/v1/chat/new` (first message) |
| **Inbound** (iMessage → Hermes) | Webhook `POST` | `/bluebubbles-webhook` on gateway webhook port, with `?password=<pwd>` query param |

- **1-on-1 messages:** Respond automatically (no mention required). Always look up the sender in `contacts-management` skill first to determine role and tone.
- **Group chats:** Only respond when mentioned (`@hermes agent` or `hermes`).
- **Mention gating:** Controlled by `BLUEBUBBLES_REQUIRE_MENTION` env var or config.
- **Cross-platform actions:** When a team member messages via iMessage, use their platform ID from `contacts-management` to auto-process actions (e.g., log operational updates or timesheets).
- **Related skill:** `contacts-management` — maps phone numbers/IDs to roles for inbound message handling.

---

## 4. SELF-HEALING DIAGNOSTIC ROUTINE

If incoming iMessages are not triggering agent responses, execute this diagnostic flow:

### Step 1 — Check Active Webhooks in BlueBubbles
```bash
curl -s "http://127.0.0.1:8645/api/v1/webhook?password=<BLUEBUBBLES_PASSWORD>"
```

### Step 2 — Clean Stale / Misconfigured Webhooks
If any webhooks point to `localhost`, lack the `/bluebubbles-webhook` path, or miss `?password=...`, delete them:
```bash
curl -X DELETE "http://127.0.0.1:8645/api/v1/webhook/<WEBHOOK_ID>?password=<BLUEBUBBLES_PASSWORD>"
```

### Step 3 — Register Correct Webhook Payload
`POST` to register the webhook with the correct URL and events:
```bash
curl -X POST "http://127.0.0.1:8645/api/v1/webhook?password=<BLUEBUBBLES_PASSWORD>"   -H "Content-Type: application/json"   -d '{
    "url": "http://127.0.0.1:8646/bluebubbles-webhook?password=<BLUEBUBBLES_PASSWORD>",
    "events": ["new-message", "updated-message"]
  }'
```

### Step 4 — Verify Gateway Service
Restart the gateway to ensure port `8646` is listening and ready:
```bash
hermes gateway restart
```

### Step 5 — Confirm Webhook is Listening
```bash
curl -s "http://127.0.0.1:8646/bluebubbles-webhook"
# Expected: "ok" or standard webhook response

lsof -i :8646  # Should show python3.1 (gateway) listening
```

---

## 5. TROUBLESHOOTING: SEND FAILURES

### Symptom: `curl` / API request times out (30–60s)
First message to a new contact creates the chat via `/api/v1/chat/new`, which routes through Apple's iMessage servers. This is normal — a timeout doesn't mean failure.
- **Verify delivery:** Check the recipient's phone or BlueBubbles UI for the message.
- **Do NOT rely on `/api/v1/chat/all`:** It is a local cache that lags behind actual iMessage delivery and may show `0` chats even after successful sends.
- **Source of truth:** Gateway logs (`~/.hermes/logs/gateway.log`) or direct confirmation from recipient.

### Symptom: `401 Unauthorized`
Password mismatch, missing password, or special characters in password being misinterpreted by the shell.
- **Fix:** Verify `BLUEBUBBLES_PASSWORD` matches the BlueBubbles server password exactly (alphanumeric only).
- **If using special characters:** They must be URL-encoded in `curl` (`!` → `%21`, `#` → `%23`, `@` → `%40`).
- **Best practice:** Use an alphanumeric-only password to avoid all encoding issues.

### Symptom: `A message is required when creating chats on macOS Big Sur or newer!` (HTTP 500)
The `/api/v1/chat/new` endpoint requires a `message` field in the JSON body on macOS Big Sur+. Using only `text` will fail.
- **Fix:** Include both fields or the required `message` field in the request body:
  ```json
  {
    "message": "Your message text here",
    "addresses": ["+15551234567"]
  }
  ```

### Symptom: `The addresses field is required.` (HTTP 400)
Using `user` instead of `addresses` in the request body.
- **Fix:** Use `addresses` (array) not `user` (string):
  ```json
  {
    "message": "Your message text here",
    "addresses": ["+15551234567"]
  }
  ```

### Symptom: `Chat not found`
The target number or email has no existing chat GUID in BlueBubbles.
- **Fix:** Hermes auto-creates chats on first message via `/api/v1/chat/new`. If this fails, check BlueBubbles logs for iMessage delivery errors.

---

## 6. HEALTH CHECKS

Add to your gateway health check cron job:

```bash
# Check BlueBubbles server:
curl -s --connect-timeout 5 "http://127.0.0.1:8645/" | head -1

# Check webhook endpoint:
curl -s --connect-timeout 5 "http://127.0.0.1:8646/health"

# Check gateway logs:
grep "bluebubbles" ~/.hermes/logs/gateway.log | tail -5
# Expected: "✓ bluebubbles connected"
```

---

## 7. COMMON PITFALLS

1. **Webhook port = server port:** Always set `BLUEBUBBLES_WEBHOOK_PORT` to a different value. This is the #1 cause of silent inbound message failure.
2. **Port conflicts:** Always verify ports are free with `lsof -i :<port>` before configuring. Common conflicts: LM Studio (often 1234), other local background services.
3. **iMessage send latency:** First messages to new contacts take 30–60s. Don't assume failure on timeout.
4. **Private API required:** BlueBubbles needs `private_api: true` and `helper_connected: true` for full functionality (tapbacks, read receipts). Check in **BlueBubbles → Settings → Advanced**.
5. **`localhost` vs `127.0.0.1`:** macOS resolves `localhost` to IPv6 (`::1`) but Hermes binds to IPv4 (`127.0.0.1`). Always use `127.0.0.1` — using `localhost` causes `ECONNREFUSED`.
6. **Password special characters:** Characters like `!`, `#`, `@` in the BlueBubbles API password cause silent failures. `#` acts as an anchor fragment (strips the rest of the password in URLs), `!` and `@` break shell history expansion. **Use alphanumeric-only passwords.**
7. **Webhook URL must include password query param:** The webhook endpoint on the BlueBubbles side must be `http://127.0.0.1:8646/bluebubbles-webhook?password=<pwd>` — without the `?password=...` query param, Hermes won't authenticate the incoming webhook and will reject it.
8. **Chat list is NOT a delivery confirmation:** `/api/v1/chat/all` returns `0` chats even after successful sends — it's a local cache that lags behind actual iMessage delivery. Never use the chat list to verify sends. Source of truth: gateway logs (`~/.hermes/logs/gateway.log`) or recipient confirmation.
