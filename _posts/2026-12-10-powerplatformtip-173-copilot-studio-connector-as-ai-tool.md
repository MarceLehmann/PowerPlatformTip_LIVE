---
title: "#PowerPlatformTip 173: 'Let Copilot Studio Build the Call - Your Connector as an AI Tool'"
date: 2026-08-27
categories:
  - Article
  - PowerPlatformTip
tags:
  - CopilotStudio
  - PowerAutomate
  - CustomConnectors
  - AI
  - MCP
  - PowerPlatformTip
excerpt: "Hand your generic Custom Connector to a Copilot Studio agent as a tool and let generative orchestration pick the endpoint and build the body itself, MCP-style. The key: it's a tool plus an instruction, not Knowledge."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
---

> **TL;DR:** Add your generic Custom Connector (#172) to a Copilot Studio agent under **Tools > Add a tool > Connector**. Generative orchestration then reads the OpenAPI definition and **builds the call itself** - picking the endpoint and filling `relativePath` + body from the user's intent. Think of it as a lightweight MCP: one tool exposes the whole API. The credential stays on the connection, so the agent never sees the key. The catch: it's a **tool + an instruction**, not Knowledge.

Once your API lives in a Custom Connector ([#PowerPlatformTip 172](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-172-generic-custom-connector-dynamic-http-request/)), the real payoff arrives: give it to a Copilot Studio agent as a **tool** and let the AI find and call the right endpoint on its own. In the new agent experience you no longer wire a separate action per endpoint - you hand over the whole definition once and generative orchestration assembles each call from the conversation. It behaves like a hand-rolled MCP server for any API that doesn't ship one, and because the API key sits on the connection, the agent authenticates automatically without ever touching the secret.

## 💡 Challenge
You want an agent that answers "list my open projects" or "how many hours did I log this week" by calling the right MOCO endpoint on its own. Wiring a separate agent action for every endpoint is tedious and never keeps up with the API. And there's a conceptual trap: people drop the API definition into **Knowledge** and wonder why the agent can't actually *call* anything.

## ✅ Solution
Add the connector **once** as a tool, invest in **descriptions**, and let generative orchestration select the operation and fill the parameters from the user's intent. Steer the edge cases with a short **agent instruction** - not with Knowledge, which is passive lookup and can't execute. One tool, every endpoint, zero per-action wiring. If the service already ships an **MCP** server, prefer that; this connector pattern is the fallback when it doesn't.

## 🔧 How it's done

**1. Expose the connector to the agent as a tool.**

🔸 In Copilot Studio: **Tools > Add a tool > Connector** and pick your custom connector. It needs **view and share permissions for the organization** so the agent can use it.
🔸 Import the MOCO **OpenAPI spec** - typed, per-entity operations beat one blind `GenericRequest`.
🔸 Write **strong descriptions** on every operation and parameter, e.g. `relativePath: MOCO endpoint, e.g. projects.json for projects, activities for time entries`. The AI decides purely from operation IDs, parameter names and these descriptions.

**2. Steer it with an agent instruction (not Knowledge).**

🔸 Add a short **instruction** that references the tool with `/`, e.g. *"When the user asks about MOCO time or projects, use /MocoRequest, build relativePath and body from the endpoints below, and confirm before any write."*
🔸 You can let the **tool** carry the typed OpenAPI definition, or paste the relevant **endpoint list** straight into the instruction (handy for a single generic call). Keep it lean - instructions have a size budget, so for large APIs favor the tool's schema over pasting everything.
🔸 Don't add the API definition as a **Knowledge** source: Knowledge is passive lookup, it can't execute or hold behavior. The tool + instruction pair is what actually drives and builds the call.

**3. Let generative orchestration do the rest.**

🔸 Enable **generative orchestration** so the agent selects the tool and fills its inputs from the conversation, with no manual mapping.
🔸 Test in the **Test your agent** pane: ask in natural language and watch it choose the operation and build the path/body.

**4. Make it robust (next step).**

🔸 Bound *directly*, a failing call (404/422) aborts before the model can react. To let the agent recover and retry, wrap the call in a flow - that's [#PowerPlatformTip 174](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-174-copilot-studio-self-healing-connector-actions/).

## 🎉 Result
The agent picks the endpoint and builds the body from one shared definition. New API areas need **no** new action, just better descriptions. Auth stays on the connection, and the whole thing behaves like a hand-rolled MCP for APIs that don't ship one.

## 🌟 Key Advantages

🔸 **AI builds the call:** path and body come from user intent; the header/auth is already fixed on the connection.

🔸 **MCP-style discovery without MCP:** one tool exposes the full API definition and the agent picks the right operation itself - the fallback when the service has no MCP server.

🔸 **Set the key once:** the connection holds the credential, the agent never re-enters, hand-sends or fetches it per call.

🔸 **No per-endpoint wiring:** descriptions, not actions, drive coverage, so the tool scales with the API.

🔸 **Ready to self-heal:** pair it with the flow wrapper in [#PowerPlatformTip 174](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-174-copilot-studio-self-healing-connector-actions/) for retry-and-remember behavior.

## 🛠️ FAQ

**Q1: How does the AI know which endpoint to call?**

From the OpenAPI operation IDs, parameter names and, most importantly, your descriptions. Vague descriptions lead the AI to the wrong endpoint or a malformed body, so invest in them.

**Q2: Should I add the API definition as Knowledge or as a tool?**

As a **tool**. The tool already carries the definition (schema + descriptions) and can execute it. Knowledge is passive reference only - it can't run the call or hold "when active, do X" behavior. Put that behavior in an **agent instruction** that references the tool with `/`.

**Q3: Should I use this or an MCP server?**

If the service offers an MCP server, prefer it - add it under **Tools > Add a tool > Model Context Protocol** and the agent auto-discovers every endpoint. This custom-connector pattern is the fallback when there is no MCP: you load the full API definition into one tool and generative orchestration selects and builds the call the same way.

**Q4: Does "authenticate once" really hold for every user?**

Only if the tool uses a **maker-provided (shared) connection**. With user-provided connections, each end user is prompted to create their own once. Either way, nobody ever re-enters the key per call.

**Q5: What happens when the call fails, and what about payload limits?**

Bound directly, a non-2xx becomes `connectorRequestFailure` **before** the LLM sees the body, so it can't self-heal - wrap it in a flow ([#PowerPlatformTip 174](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-174-copilot-studio-self-healing-connector-actions/)). Also note Copilot Studio caps connector payloads (5 MB public cloud; 450 KB GCC), so filter large MOCO lists with query/limit inputs.

## 🔗 Related Tips
- [#PowerPlatformTip 172: One Custom Connector, Any Endpoint](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-172-generic-custom-connector-dynamic-http-request/), the connector this tip turns into an AI tool.
- [#PowerPlatformTip 174: Self-Healing Connector Actions in Copilot Studio](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-174-copilot-studio-self-healing-connector-actions/), wrap the call so the agent retries and remembers the fix.
- [#PowerPlatformTip 170: One Flow, Many Tools - a Switch Router for Copilot Studio Agents](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-170-flow-as-agent-tool-switch-router/), the internal, no-connector way to give an agent a tool.
- [#PowerPlatformTip 138: Graph API via HTTP with Microsoft Entra ID](https://www.powerplatformtip.com/article/powerplatformtip/PowerPlatformTip-138-Graph-API-HTTP/), central auth without an app registration.

---

> 📌 **Scheduling note:** I moved this tip forward in the publishing schedule - the rest of this Copilot Studio series (and some of the related tips referenced above, e.g. #172 and #174) are published later and will follow in the coming weeks.
