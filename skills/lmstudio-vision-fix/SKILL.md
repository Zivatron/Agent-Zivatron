---
name: lmstudio-vision-fix
description: Troubleshoot and fix LM Studio vision payload errors (HTTP 400 'Invalid messages') when native vision or MLX models are enabled in Hermes Agent.
author: Community
version: 1.0.0
license: MIT
---

# Installation & Setup Configuration

> **Note for Users Installing This Skill:**  
> Apply this skill whenever running local OpenAI-compatible inference backends (such as LM Studio, MLX, or vLLM) with native vision capabilities enabled, particularly when experiencing HTTP 400 errors during multimodal tool calls.

---

# SKILL DEFINITION: LM STUDIO VISION FIX

## 1. WHEN TO USE

Use this procedure when the agent encounters an **HTTP 400 error** after a vision or image processing tool call completes (e.g., `vision_analyze`), returning errors such as:
- `"Invalid 'messages' in payload"`
- `"model provider failed"`
- `"HTTP 400 Bad Request"` during post-tool-call completion steps.

---

## 2. DIAGNOSIS

1. **Check Agent Gateway Logs:**
   ```bash
   grep -i "Invalid.*messages\|vision" ~/.hermes/logs/agent.log | tail -20
   ```
   *Look for the pattern where the vision tool call succeeds, but the subsequent LLM completion request fails with an HTTP 400 status.*

2. **Verify Configuration:**
   Confirm that `supports_vision: true` is set in your Hermes `config.yaml`.

---

## 3. FIX

Set `agent.image_input_mode` to `native`. This forces images to attach directly to the user message turn rather than getting wrapped inside an intermediate tool response payload:

```bash
hermes config set agent.image_input_mode native
```

Restart the gateway to apply the configuration:

```bash
hermes gateway restart
```

---

## 4. WHY THIS WORKS

When `supports_vision: true` is enabled, Hermes routes image payloads natively. After a tool call completes, Hermes reconstructs the conversation messages array with the image attached and sends it back to the local provider. 

Certain local server API layers (such as LM Studio's OpenAI-compatible endpoint) reject this specific payload structure during tool-response transitions. Setting `image_input_mode: native` attaches the image directly to the primary user message turn instead, bypassing the payload transition bug.

---

## 5. ALTERNATIVE WORKAROUND (NOT RECOMMENDED)

Disable `supports_vision: true` in your configuration. This causes Hermes to route images through an auxiliary vision model to generate a text description. 

> **Warning:** Disabling native vision removes direct visual reasoning capability from the primary model and relies solely on textual descriptions.
