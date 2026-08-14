---
title: "#PowerPlatformTip 170: 'One Flow, Many Tools – a Switch Router for Copilot Studio Agents'"
date: 2026-08-20
categories:
  - Article
  - PowerPlatformTip
tags:
  - PowerAutomate
  - CopilotStudio
  - Agent
  - MCP
  - Switch
  - PowerPlatformTip
excerpt: "Give a Copilot Studio agent a single instant flow that acts like a mini tool server. One 'action' input drives a Switch (get / create / update), one generic 'input' carries either an OData filter or a JSON body, and 'Respond to a PowerApp or flow' returns clean JSON back to the agent."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
---

> **TL;DR:** Build one manually triggered flow with an `action` and an `input` parameter. A Switch on `action` routes to *get* (OData filter), *create* or *update* (JSON body), and *Respond to a PowerApp or flow* returns JSON. Add it to a Copilot Studio agent as a tool, no premium HTTP endpoint or custom connector needed.

Everyone talks about turning Power Automate flows into MCP servers, but a real MCP server needs a premium HTTP trigger, JSON-RPC handling and a custom connector. For an **internal** agent you don't need any of that. A plain instant (button) flow added as an **agent flow tool** gives Copilot Studio the same value: the flow name becomes the tool name, its trigger inputs become the parameters, and *Respond to a PowerApp or flow* returns the result. This tip packs several operations into **one** flow using a Switch router.

## 💡 Challenge
You want a Copilot Studio agent to read and write your data (Dataverse, SharePoint, …) through Power Automate. Building a proper MCP server means a premium `When a HTTP request is received` trigger, JSON-RPC methods like `tools/list` and `tools/call`, and a custom connector with `x-ms-agentic-protocol: mcp-streamable-1.0`. That's a lot of plumbing for an internal scenario, and it needs premium licensing.

## ✅ Solution
Use a single **instant flow** (*Manually trigger a flow*) and expose it to the agent as a tool. Drive the behaviour with two inputs:

- **`action`** – which operation to run: `get`, `create` or `update`.
- **`input`** – a generic string whose meaning depends on the action: an **OData filter** for `get`, a **JSON body** for `create` / `update`.

A **Switch** on `action` routes to the right branch, and one *Respond to a PowerApp or flow* action returns JSON. Copilot Studio handles all the tool discovery, so you never touch JSON-RPC yourself.

## 🔧 How it's done

**1. Create the trigger inputs.** On *Manually trigger a flow*, add:

🔸 `action` (Text) – describe it for the model: *"get, create or update"*.

🔸 `input` (Text) – *"For get: an OData filter like `Status eq 'open'`. For create/update: a JSON object like `{\"title\":\"Test\"}`."*

🔸 `id` (Text, optional) – *"Record ID, required for update"*.

Clear input descriptions matter: the agent reads them to decide what to pass.

**2. Add a Switch** on the action value.

```
triggerBody()?['text']
```

🔸 The exact reference is `['text']`, `['text_1']`, `['text_2']` depending on input order, check the *See more* / peek code view.

**3. Case `get` – treat `input` as OData.** Use *List rows* / *Get items* and drop the string straight into **Filter Query**:

```
triggerBody()?['text_1']
```

🔸 No parsing needed, an OData filter is already a plain string.

**4. Case `create` – treat `input` as JSON.** Add *Parse JSON* on `triggerBody()?['text_1']` with a schema for your fields:

```json
{ "type": "object", "properties": {
    "title":  { "type": "string" },
    "status": { "type": "string" }
} }
```

🔸 Then feed `body('Parse_JSON')?['title']` etc. into *Add a row* / *Create item*.

**5. Case `update` – JSON body plus an ID.** Same *Parse JSON* as above, target the record with:

```
triggerBody()?['text_2']
```

🔸 Skip empty fields gracefully with `coalesce(body('Parse_JSON')?['status'], null)`.

**6. Respond with JSON.** After the Switch, add a *Compose* and return it via *Respond to a PowerApp or flow* (a `result` text output):

```json
{
  "ok": true,
  "action": "@{triggerBody()?['text']}",
  "result": @{coalesce(outputs('List_rows')?['body/value'], body('Parse_JSON'))}
}
```

🔸 Use `@{...}` for string values and raw `@{body(...)}` for nested JSON.

**7. Handle bad JSON.** Wrap *Parse JSON* in a **Scope** and use **Configure run after** on a fallback Respond to return `{"ok":false,"error":"invalid JSON"}` when a create/update payload isn't valid JSON.

**8. Add it to the agent.** Put the flow in a **solution**, then in Copilot Studio open your agent → **Tools → Add a tool → Flow** and select it. The flow name becomes the tool name, and the `action` / `input` / `id` inputs become the tool parameters.

## 🎉 Result
Your agent gets one dependable tool that can read (OData), create and update records, all from a single standard flow. No premium HTTP trigger, no JSON-RPC, no custom connector, and everything stays inside your governed Power Platform environment.

## 🌟 Key Advantages

🔸 **No premium plumbing:** an instant flow as an agent tool replaces the whole HTTP + JSON-RPC + custom connector stack for internal use.

🔸 **One tool, many operations:** the Switch on `action` keeps `get` / `create` / `update` in a single, maintainable flow.

🔸 **Flexible input:** one `input` parameter carries an OData filter or a JSON body, interpreted per branch.

🔸 **Clean contract:** structured JSON responses (`ok`, `action`, `result`) make the agent's downstream handling reliable.

## 🛠️ FAQ

**Q1: Is this a real MCP server?**

No. A real MCP server needs a premium HTTP trigger, JSON-RPC (`tools/list`, `tools/call`) and a custom connector with `x-ms-agentic-protocol: mcp-streamable-1.0`. This is the **internal** equivalent: an agent flow tool where Copilot Studio does the tool discovery for you.

**Q2: One flow with a Switch, or one flow per operation?**

Both work. A Switch keeps everything in one tool. Separate flows (`create`, `get`, `update`) make each flow name a distinct tool with its own description and parameters, which the agent often selects more reliably. Choose based on how many operations you have.

**Q3: Why not parse the OData string in the `get` branch?**

An OData filter is already a plain string that goes straight into *Filter Query*. Only `create` / `update` need *Parse JSON*, because you read individual fields out of the body.

**Q4: How does the agent know what to send?**

From your input descriptions. Spell out the expected format (OData vs. JSON) in each trigger input's description, that's the text the model uses to build the call.

## 🔗 Related Tips
- [#PowerPlatformTip 158: Return Flow Data to Power Apps with ParseJSON](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-158-parsejson-return-flow-data-power-apps/), the same *Respond to a PowerApp or flow* + JSON pattern this router returns.
- [#PowerPlatformTip 104: Efficient JSON Handling](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-104-efficient-json-handling/), Parse JSON schemas for the `create` / `update` branches.
- [#PowerPlatformTip 95: Optimized SharePoint Queries](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-95-optimized-sharepoint-queries/), building the OData Filter Query used in the `get` branch.
- [#PowerPlatformTip 118: Copy Actions in Switch/Condition](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-118-copy-actions-in-switch_condition/), working cleanly inside Switch branches.
