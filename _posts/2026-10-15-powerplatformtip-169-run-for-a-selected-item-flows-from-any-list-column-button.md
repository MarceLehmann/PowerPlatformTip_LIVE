---
title: "#PowerPlatformTip 169: 'Run a For-a-Selected-Item Flow from Any List with a Column Button'"
date: 2026-09-10
categories:
  - Article
  - PowerPlatformTip
tags:
  - PowerAutomate
  - SharePoint
  - ColumnFormatting
  - Governance
  - PowerPlatformTip
excerpt: "For a selected item only runs from the Default environment, and many companies won't allow productive flows there. Don't switch to the premium HTTP trigger: keep the standard For-a-selected-item trigger, launch it from a column-formatting button with customRowAction executeFlow, read the entity (ID, itemUrl, fileName) plus the header (who triggered, from where), and one flow serves any list."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
---

> **TL;DR:** *For a selected item* only runs from the **Default** environment, which many companies forbid for productive flows. Instead of switching to the **premium** HTTP trigger, keep the **standard** *For a selected item* trigger and start it from a **column-formatting button** with `customRowAction: executeFlow`. The flow receives an `entity` (`ID`, `itemUrl`, `fileName`, `FileId`) and the request **header** (who triggered, from where), and one flow serves **any** list.

*For a selected item* is the classic way to let users pick a row and start a flow, but it **only works from the Default environment**. Many organizations lock Default down (no productive apps or flows), so the moment you move the flow to a productive environment the *Integrate → Power Automate* command-bar button vanishes. You could rebuild it with a *When an HTTP request is received* trigger, but that's a **premium** connector. The better answer: keep the **standard** *For a selected item* trigger and simply launch it yourself from a **column button**.

## 💡 Challenge
*For a selected item* only runs from the Default environment, and many companies won't allow productive flows there. Once the flow moves to a productive environment, the native command-bar button is gone. You still want a *"click a button on this row"* experience, but **without** paying for the premium HTTP trigger, and ideally with **one** flow across lists, not a copy per list.

## ✅ Solution
Two things make this work:

🔸 **The trigger's Site/List selection is irrelevant.** *For a selected item* fires from wherever it's launched and passes the real context at runtime through its **`entity`** output. Whatever site or list you pick in the trigger config, it still runs for the row the user clicked.

🔸 **Launch it from a column button.** SharePoint **column formatting** supports `customRowAction: executeFlow`, which starts a specific flow by **ID** for the current row, no command-bar button, no Default-environment dependency.

Inside the flow you receive:

```json
{
  "body": {
    "entity": {
      "ID": 141,
      "itemUrl": "https://contoso.sharepoint.com/teams/2147/Shared Documents/Anna Keller",
      "fileName": "Anna Keller",
      "FileId": "141"
    }
  }
}
```

From `itemUrl` you can derive the **site** and **library/list**, and `ID` is the list item ID you use for *Get item*. Pass site + list + ID onward and pull every other field from the item.

## 🔧 How it's done

**1. Build the flow with the *For a selected item* trigger.**

🔸 Add **For a selected item** (or **For a selected file**).

🔸 Pick any Site Address and List in the trigger, it only needs a valid selection to save; the real target comes from `entity` at runtime.

**2. Read the item context from `entity`.**

🔸 The trigger returns `ID`, `itemUrl`, `fileName` and `FileId`. Grab the ID with:

```
triggerBody()?['entity']?['ID']
```

🔸 Derive the **site URL** from `itemUrl` (everything up to and including the site, e.g. `.../teams/2147`):

```
first(split(triggerBody()?['entity']?['itemUrl'], '/Shared Documents'))
```

🔸 Now call **Get item** (Site Address = derived site, List/Library = your target, Id = the `ID`) and read every other field from there.

🔸 Want to know **who** started it and **from where**? The trigger also exposes the request header, read the caller's email and source with:

```
triggerOutputs()?['headers']?['x-ms-user-email']
```

**3. Get the flow ID for the button, straight from the flow URL.**

The `executeFlow` action needs the flow ID in the form `v1/<environment-id>/<flow-guid>`. You can read **both** parts from the flow's URL in Power Automate. Open the flow and look at the address bar:

```
https://make.powerautomate.com/environments/Default-16b0ff2b-bf0e-e4ad-afc0-1d4d2911e350/flows/7503af2b-1cf8-4e3c-a7ff-dbac9ae240f4/details
```

🔸 **Environment ID** = the value after `/environments/`, **without** the `Default-` prefix:
`Default-16b0ff2b-bf0e-e4ad-afc0-1d4d2911e350` → `16b0ff2b-bf0e-e4ad-afc0-1d4d2911e350`

🔸 **Flow GUID** = the value after `/flows/`:
`7503af2b-1cf8-4e3c-a7ff-dbac9ae240f4`

🔸 **Assemble** them as `v1/<environment-id>/<flow-guid>`:

```
v1/16b0ff2b-bf0e-e4ad-afc0-1d4d2911e350/7503af2b-1cf8-4e3c-a7ff-dbac9ae240f4
```

That is exactly the string you drop into `actionParams` in the next step. (Alternatively, add the flow to the list once via *Integrate → Power Automate* and copy the `id` it generates, same value.)

**4. Add a button column with `customRowAction: executeFlow`.**

🔸 Create a column (e.g. single line of text) named **Run** → **Column settings → Format this column → Advanced mode**, and paste:

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/sp/v2/column-formatting.schema.json",
  "elmType": "button",
  "txtContent": "▶ Run",
  "customRowAction": {
    "action": "executeFlow",
    "actionParams": "{\"id\":\"v1/16b0ff2b-bf0e-e4ad-afc0-1d4d2911e350/7503af2b-1cf8-4e3c-a7ff-dbac9ae240f4\"}"
  },
  "style": {
    "border": "none",
    "background-color": "#2dd4bf",
    "color": "white",
    "padding": "4px 10px",
    "border-radius": "4px",
    "cursor": "pointer"
  }
}
```

🔸 Replace the `id` inside `actionParams` with **your** assembled value from step 3. Keep the `v1/` prefix and the two GUIDs separated by a single `/`.

**5. Test it.**

🔸 Click **▶ Run** on a row. The flow starts for that item, and its run history shows the `entity` with the clicked row's `ID` and `itemUrl`.

🔸 Reuse the exact same JSON on another list, no new flow needed. The `entity` tells the flow which item was clicked.

## 🎉 Result
Users keep the familiar per-row button, but the flow lives wherever governance requires it. Because the trigger's own Site/List selection is ignored and the real context arrives via `entity`, a **single** flow serves many lists across sites.

## 🌟 Key Advantages

🔸 **No premium:** *For a selected item* is a standard trigger, unlike *When an HTTP request is received*, which is premium.

🔸 **List-agnostic:** the trigger's Site/List config doesn't matter, `entity` carries the real item.

🔸 **One flow, many lists:** paste the same `executeFlow` JSON everywhere; the ID points at one flow.

🔸 **Full context:** the request header tells you who clicked and from where (`x-ms-user-email`), no extra plumbing.

🔸 **Native feel:** a real per-row button inside the list via column formatting.

## 🛠️ FAQ

**Q1: Does the Site Address I pick in the trigger really not matter?**

Correct. *For a selected item* passes the actual item at runtime in `entity` (`ID`, `itemUrl`, `fileName`, `FileId`). The trigger's configured site/list just needs to be a valid selection so the flow saves, execution uses the clicked row.

**Q2: How do I turn `itemUrl` into a usable Site Address for *Get item*?**

Split `itemUrl` on the library segment (e.g. `/Shared Documents`) and take the first part, that's your site URL. Then use *Get item* / *Get file properties* with that site, your list name and the `ID`.

**Q3: Where exactly in the flow URL are the two IDs?**

Open the flow and read the address bar: `.../environments/Default-<ENV-GUID>/flows/<FLOW-GUID>/details`. The **environment ID** is the GUID after `/environments/` (drop the `Default-` prefix), the **flow GUID** is the part after `/flows/`. Combine them as `v1/<ENV-GUID>/<FLOW-GUID>` and paste that into `actionParams`.

**Q4: Does this also work for documents?**

Yes, use **For a selected file** and *Get file properties*. You still get `ID`/`FileId` and `itemUrl` in `entity`.

**Q5: Why not just use the *When an HTTP request is received* trigger?**

That trigger is a **premium** connector. *For a selected item* is standard, so the column-button approach keeps the solution premium-free while still giving you the row context (via `entity`) and the caller (via the header).

**Q6: Can I keep the flow as a governed exception instead of rebuilding?**

Yes. If it's just a few low-risk helpers, whitelisting them (service-account owner, documented, DLP-scoped) is a valid alternative to the column-button redesign.

## 🔗 Related Tips
- [#PowerPlatformTip 135: One Flow, Many Users](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-135-One-Flow-Many-Users/), one flow as a shared trigger hub for the whole company.
- [#PowerPlatformTip 34: PowerApps V2](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-34-powerapps-v2/), typed inputs and a service-user connection for reusable, governed flows.
- [#PowerPlatformTip 21: Use Triggeroutput](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-21-use-triggeroutput/), read request/header data to identify and notify the caller.
