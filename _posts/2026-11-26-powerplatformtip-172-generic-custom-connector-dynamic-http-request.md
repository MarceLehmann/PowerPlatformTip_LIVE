---
title: "#PowerPlatformTip 172: 'One Custom Connector, Any Endpoint - Central Auth with a Dynamic HTTP Request'"
date: 2026-10-01
categories:
  - Article
  - PowerPlatformTip
tags:
  - PowerAutomate
  - CustomConnectors
  - HTTP
  - API
  - CopilotStudio
  - Governance
  - PowerPlatformTip
excerpt: "Build one generic Custom Connector that authenticates once and calls any endpoint of an API (MOCO example) by passing the relative path dynamically. Share the connection, reuse everywhere, and hand the whole definition to Copilot Studio as a tool."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
---

> **TL;DR:** Build one Custom Connector that stores the API key on the *connection* and exposes a single generic action taking the HTTP method, a `relativePath` and a body. Authenticate once, share the connection, and reach every endpoint of a token-based REST API (e.g. MOCO) without ever re-entering the key. Bonus: hand the whole definition to Copilot Studio as a tool.

Instead of building a separate action for every endpoint, create one Custom Connector that holds the authentication centrally and exposes a single generic action where the caller passes the HTTP method, relative path and body at runtime. You build the connector once, create one connection with the credentials, and from then on every flow, app or agent just reuses it dynamically - the secret stays on the connection and out of your flow logic. The real payoff comes later: once the definition lives in a connector, you can hand it to Copilot Studio as a tool. The catch: Data Loss Prevention (DLP) applies to the whole connector, not per endpoint, so treat it as a governed building block.

## 💡 Challenge
You want to talk to many endpoints of the same REST API (e.g. MOCO: `projects.json`, `activities`, `users`) from Power Automate or Power Apps. Building and maintaining one action per endpoint is tedious, and repeating the API key in every raw HTTP action - pasting it into a header or pulling it from a vault on every call - is a security and maintenance nightmare.

## ✅ Solution
Create a Custom Connector that separates the two things a connector really owns: the **connection** (host + security scheme) and the **definition** (one reusable action). Configure the API key **once** on the connection, then send only the **relative path** and **body** per call. The key is stored, encrypted, on the connection, so it never appears in your flows again.

## 🔧 How it's done

**1. General tab - set the host.**

🔸 Host: `yourcompany.mocoapp.com`, **Base URL**: `/api/v1`.

**2. Security tab - authenticate once.**

🔸 Authentication type: **API Key**.
🔸 Parameter label: `Authorization`, location: **Header**.
🔸 MOCO expects the value `Token token=YOUR_API_KEY`, so the connection user pastes exactly that when creating the connection. The header is then attached to *every* call automatically.

**3. Definition tab - one generic action.**

🔸 Add an action, e.g. `GenericRequest`.
🔸 Import from sample: pick a verb (e.g. `GET`) and a sample URL like `https://yourcompany.mocoapp.com/api/v1/projects.json`.
🔸 Turn the path segment into a **string parameter** (e.g. `relativePath`) so the caller decides `projects.json`, `activities`, `users`, ... at runtime.
🔸 Add optional parameters: **query** parameters and a **body** (for `POST`/`PUT`).
🔸 Write **clear descriptions** on every parameter - they become the connector's self-documentation and are exactly what Copilot Studio reads later.

**4. Use it in a flow.**

🔸 Add the connector action, set `relativePath` to e.g. `activities`, method to `POST`, and pass a JSON body. The `Authorization` header is already there from the connection - you never touch the key again.

## 🎉 Result
One connector, one shared connection, every endpoint reachable. New API areas need **no** connector change, you just pass a different `relativePath`. Auth lives in one place and is distributable through a Solution.

## 🌟 Key Advantages

🔸 **Set the key once, forget it:** the API key lives on the connection. You never re-enter it, never hand-build an auth header, and never fetch it from a vault in each flow.

🔸 **Auth decoupled from logic:** whoever builds flows or apps just picks the connection, no secret ever touches the flow definition.

🔸 **Future-proof:** new endpoints work instantly, no redeploy, just a different `relativePath`.

🔸 **Shareable & ALM-ready:** package the connector in a Solution and share the connection across the team.

🔸 **A ready-made Copilot Studio tool:** the connector definition can be attached to an agent as a tool, so the AI browses the endpoints and builds the call itself.

## 🛠️ FAQ

**Q1: Generic action vs. importing the OpenAPI spec, which is better?**

Generic gives maximum flexibility with minimal setup; OpenAPI import gives typed schemas and IntelliSense-like parameters per endpoint. MOCO ships a machine-readable OpenAPI spec, and for AI-driven agents (see #173) the typed actions win.

**Q2: Do I have to expose the API key in every flow?**

No. That's the whole point - the key is stored once on the connection; flows only send path, query and body.

**Q3: When is a plain HTTP action enough instead?**

For a quick one-off internal call. As soon as multiple people or apps must reuse the same authentication, the Custom Connector wins on sharing, security and ALM.

**Q4: Can Copilot Studio really use this without extra work?**

Yes, add the connector as a **tool** (Tools > Add a tool > Connector); it needs view and share permissions for your organization. Because your parameter descriptions document each endpoint, the agent can pick the path and build the body itself. Details in [#PowerPlatformTip 173](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-173-copilot-studio-connector-as-ai-tool/).

**Q5: What's the governance catch with a generic "call anything" connector?**

DLP classification applies to the **entire** connector, not per endpoint, and is evaluated **before** any runtime host policy. A generic connector can effectively reach any REST API, so build and share it only with trusted parties and align it with your CoE before a productive rollout.

**Q6: Can I combine several authentication types, or make the OAuth URL dynamic?**

Not in the wizard. API Key, Basic and OAuth 2.0 are exclusive; multiple auth needs `connectionParameterSets` in `apiProperties.json` via the Power Platform Connectors CLI (`paconn`). And custom connection parameters aren't injected into the OAuth `/authorize` request, so the authorize URL can't be made dynamic - fine for MOCO's token header, relevant if your API uses OAuth 2.0.

## 🔗 Related Tips
- [#PowerPlatformTip 173: Let Copilot Studio Build the Call - Your Connector as an AI Tool](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-173-copilot-studio-connector-as-ai-tool/), use this connector as an AI tool.
- [#PowerPlatformTip 174: Self-Healing Connector Actions in Copilot Studio](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-174-copilot-studio-self-healing-connector-actions/), wrap the call so the agent retries and remembers the fix.
- [#PowerPlatformTip 58: HTTP Actions](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-58-http-actions/), the raw-HTTP alternative for quick one-offs.
- [#PowerPlatformTip 138: Graph API via HTTP with Microsoft Entra ID](https://www.powerplatformtip.com/article/powerplatformtip/PowerPlatformTip-138-Graph-API-HTTP/), central auth without an app registration.
- [#PowerPlatformTip 61: Power of Environment Variables](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-61-power-of-environment-variables/), parameterize host/keys across environments for ALM.
