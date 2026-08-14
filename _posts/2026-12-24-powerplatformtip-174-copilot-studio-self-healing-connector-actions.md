---
title: "#PowerPlatformTip 174: 'Self-Healing Connector Actions in Copilot Studio - Retry & Remember'"
date: 2026-09-03
categories:
  - Article
  - PowerPlatformTip
tags:
  - CopilotStudio
  - PowerAutomate
  - CustomConnectors
  - AI
  - ErrorHandling
  - Memory
  - PowerPlatformTip
excerpt: "Bind a connector directly and a 404/422 hard-fails before the AI can react. Wrap the call in a flow that always returns readable JSON, so the agent retries in a capped loop and, with Memory, remembers the fix for next time."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
---

> **TL;DR:** A connector bound *directly* as a Copilot Studio tool treats any non-2xx status as a **hard failure** - the LLM never sees the error, so it can't self-correct. Wrap the call in a **flow** that catches every status and always returns clean JSON (`ok`, `status`, `error`, `data`). The agent reads that, corrects its parameters and retries in a **capped loop**, and with **Memory (preview)** it remembers the winning fix for next time.

You gave your Custom Connector to a Copilot Studio agent as a tool and let generative orchestration build the call ([#PowerPlatformTip 173](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-173-copilot-studio-connector-as-ai-tool/)). Now make it survive real-world API errors. Real APIs return 400/404/422 for a wrong path or a malformed body - but a directly bound connector action aborts on those *before* the model can react, so it can neither fix itself nor explain what went wrong. A thin flow wrapper turns every error into feedback the agent can act on, and Memory lets it carry the correction into future conversations.

## 💡 Challenge
You want an agent that answers "log 2 hours on project X" by calling the right MOCO endpoint on its own. But a directly bound connector action aborts on 400/404/422 with `connectorRequestFailure` **before** the AI can react, so it can neither correct itself nor explain the problem - a dead end on the first bad path or body.

## ✅ Solution
Route the actual HTTP call through a **Power Automate flow** that catches every status and always returns a clean, structured JSON. The agent reads that JSON, corrects its parameters and tries again in a **capped loop** - true self-healing - and with **Memory** turned on it keeps the winning fix for next time. This is the connector-specific, Memory-backed sibling of the `result()` pattern from [#PowerPlatformTip 168](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-168-collect-failed-action-details-with-result/).

## 🔧 How it's done

**1. Understand the direct-binding limit.**

🔸 Bound as a generative tool, a non-2xx status becomes `connectorRequestFailure` **before** the LLM sees the body. Instructions can't override this - it happens at the connector layer, not the model.

**2. Wrap the call in a flow (the self-healing part).**

🔸 Put the connector/HTTP action inside a **Scope** (see [#PowerPlatformTip 45](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-45-use-scopes/)).
🔸 On the next action set **Configure run after** to run on *is successful*, *has failed* **and** *has timed out* (the try-catch-finally idea from [#PowerPlatformTip 15](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-15-try-catch-finally/)).
🔸 Always end with **Respond to the agent** returning a fixed shape, e.g. `{ "ok": false, "status": 422, "error": "…", "data": null }`, regardless of the real HTTP status.

**3. Add the flow as the agent's tool.**

🔸 Build the flow on the **"When an agent calls the flow"** trigger and end it with the **"Respond to the agent"** action.
🔸 In Copilot Studio: **Tools > Add a tool >** your flow. Enable **generative orchestration** so the agent selects the tool and fills inputs itself.
🔸 Because the flow never "fails", the agent always receives readable JSON and can reason: retry with a corrected path/body, or explain the problem.

**4. Retry loop, then remember the fix (Memory).**

🔸 Because the flow returns a readable error instead of failing, the agent can **reconsult the definition, rebuild and retry**, looping until it gets a 2xx. Cap it: instruct a **max number of attempts** (e.g. 3) so it doesn't loop or burn tokens forever.
🔸 Turn on **Memory (preview)** in the **Build** tab. An agent can *"apply learned patterns or corrections to future interactions"*, so once it discovers the correct path/body it stores that and stops repeating the same mistake next time.

**5. Guardrails for writes.**

🔸 For `POST`/`PUT` (e.g. creating activities) add a **confirmation step** before sending - in the flow, or via the agent's instructions - to avoid wrong bookings in MOCO.

## 🎉 Result
The agent picks the endpoint, builds the body, recovers from its own mistakes in a bounded loop, and - with Memory on - avoids repeating them, instead of dead-ending on a hard error. You keep the exact "catch any status, return it, let it retry" behavior you already trust from Power Automate.

## 🌟 Key Advantages

🔸 **Self-healing:** the flow wrapper guarantees a readable response, so the agent can retry with corrections.

🔸 **Deterministic contract:** one JSON shape for success and failure keeps the model's reasoning stable.

🔸 **Learns from mistakes:** with Memory (preview) on, a correction discovered once is applied on future calls - fewer repeat failures over time.

🔸 **Bounded:** a max-attempts rule keeps the loop, and your token spend, under control.

🔸 **Safe writes:** confirmation before `POST`/`PUT` prevents accidental records.

## 🛠️ FAQ

**Q1: Why not bind the connector directly, it's simpler?**

It is, until an endpoint returns 404/422. Then the agent hard-fails with `connectorRequestFailure` and can't recover, because the non-2xx is caught at the connector layer before the LLM sees the body. The flow wrapper is what enables self-healing.

**Q2: What exactly should the flow return?**

A fixed JSON contract, e.g. `ok`, `status`, `error`, `data`. Same shape on success and failure so the model always knows how to read it. Watch the payload cap too (5 MB public cloud; 450 KB GCC) and filter large MOCO lists with query/limit inputs.

**Q3: Can the agent remember the fix so it doesn't repeat the error?**

Yes, turn on **Memory (preview)** in the Build tab. The agent captures corrections and applies them on future interactions. It's per-user, expires after 28 days of inactivity, and is preview - so pair it with a capped retry loop rather than relying on it alone.

**Q4: How do I stop it from looping forever?**

Put a **max-attempts** rule in the agent instructions (e.g. try at most 3 times, then explain the failure). The readable JSON makes each retry productive; the cap keeps it bounded. Start with read-only `GET`s, validate, then enable writes.

## 🔗 Related Tips
- [#PowerPlatformTip 173: Let Copilot Studio Build the Call - Your Connector as an AI Tool](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-173-copilot-studio-connector-as-ai-tool/), the AI-tool setup this tip makes robust.
- [#PowerPlatformTip 168: Collect Failed Action Details with result()](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-168-collect-failed-action-details-with-result/), the general "return the error as an output so the agent self-corrects" pattern.
- [#PowerPlatformTip 172: One Custom Connector, Any Endpoint](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-172-generic-custom-connector-dynamic-http-request/), the connector behind the flow.
- [#PowerPlatformTip 15: try-catch-finally](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-15-try-catch-finally/), the error-handling backbone of the flow wrapper.
- [#PowerPlatformTip 45: Use Scopes](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-45-use-scopes/), group and isolate the HTTP call for clean run-after handling.
