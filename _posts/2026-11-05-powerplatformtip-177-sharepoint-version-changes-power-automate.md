---
title: "#PowerPlatformTip 177: 'See What Actually Changed Between SharePoint Versions with Power Automate'"
date: 2026-11-05
categories:
  - Article
  - PowerPlatformTip
tags:
  - PowerAutomate
  - SharePoint
  - REST
  - Versions
  - HTTP
  - PowerPlatformTip
excerpt: "SharePoint keeps a full version history, but the connector won't tell you what changed. One 'Send an HTTP request to SharePoint' GET against the item's /versions endpoint returns every field value per version, so you can diff the two newest versions and log exactly which fields changed, from and to which value."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
---

> **TL;DR:** Call `_api/web/lists/getbytitle('List')/items(ID)/versions` with *Send an HTTP request to SharePoint*. It returns every field value for each version (newest first). Compare `value[0]` (current) against `value[1]` (previous) field by field to get exactly what changed, old value → new value.

When an item or file is edited in SharePoint, the *When an item is created or modified* trigger fires, but it only hands you the **new** state. It never tells you **which** columns changed or what the **previous** values were. The version history holds that information; you just have to read it. The `/versions` REST endpoint returns the full field set for every retained version, so you diff the two most recent ones yourself.

## 💡 Challenge
You want an audit trail like "*Status* changed from `Draft` to `Approved`, *Amount* changed from `100` to `250`" whenever someone edits a list item or updates a document's metadata.
The SharePoint connector has no "get version differences" action. The modified trigger gives you only the current values, and *Get changes for an item or a file (properties only)* just returns a list of column names that changed, not their old and new values, and only works between two known trigger runs.

## ✅ Solution
Every list item and every file with versioning enabled exposes a **`/versions`** collection through the SharePoint REST API. Each entry contains **all field values as they were at that version**, plus `VersionLabel`, `Modified`, `IsCurrentVersion` and the editor.
Read that collection with **Send an HTTP request to SharePoint**, take the two newest entries, and compare their fields to produce a precise before/after delta, no premium connector, no app registration.

## 🔧 How it's done

**1. Trigger the flow on the change.**

🔸 Use *When an item is created or modified* (list items) or *When a file is created or modified (properties only)* (document libraries). Grab the item **ID**.

🔸 Make sure **versioning is enabled** on the list/library (List settings → Versioning settings). Without it there is no history to compare.

**2. Add *Send an HTTP request to SharePoint* to read the versions.**

🔸 **Site Address:** your site.

🔸 **Method:** `GET`

🔸 **Uri:**

```
_api/web/lists/getbytitle('Contracts')/items(@{triggerBody()?['ID']})/versions?$orderby=VersionId desc
```

🔸 **Headers:**

```
Accept: application/json;odata=nometadata
```

🔸 The response `value` array is ordered newest → oldest: `value[0]` is the current version, `value[1]` the one right before it.

**3. Read the two newest versions into variables.**

🔸 Current version object:

```
@{outputs('Send_an_HTTP_request_to_SharePoint')?['body']?['value']?[0]}
```

🔸 Previous version object:

```
@{outputs('Send_an_HTTP_request_to_SharePoint')?['body']?['value']?[1]}
```

🔸 Guard against first-ever versions with a Condition: only continue when `length(...body/value) >= 2`.

**4. Diff the fields you care about.**

🔸 For a targeted audit, compare specific columns directly. Add a *Condition*:

```
@not(equals(variables('CurrentVersion')?['Status'], variables('PreviousVersion')?['Status']))
```

🔸 If true, the value changed. Build the message with both sides:

```
Status: @{variables('PreviousVersion')?['Status']} → @{variables('CurrentVersion')?['Status']}
```

🔸 To diff **all** columns generically, run *Apply to each* over the current version's keys and compare each key's value against the same key in the previous version, collecting every key where they differ.

**5. Do something with the delta.**

🔸 Write it to an audit list, post it to Teams, or email it. You now have field name, old value and new value for every change.

🔸 For a **document library** the pattern is identical, `getbytitle('Documents')/items(ID)/versions` returns the metadata history of the file's list item.

## 🎉 Result
On every edit your flow reports exactly which columns changed and their before/after values, straight from SharePoint's own version history, using one standard HTTP call.

## 🌟 Key Advantages

🔸 **Real old-vs-new values:** unlike *Get changes*, you get the actual previous value, not just the column name.

🔸 **No premium, no Graph app registration:** *Send an HTTP request to SharePoint* is part of the standard SharePoint connector.

🔸 **Works for items and files:** same endpoint pattern for list items and document metadata.

## 🛠️ FAQ

**Q1: Why not use the *Get changes for an item or a file (properties only)* action?**

It returns only the names of columns that changed between two trigger runs and needs a stored trigger state, not the old and new values. The `/versions` endpoint gives you the full value of every field at each version, so you can build a true before/after diff.

**Q2: The `value` array only has one entry, why?**

Versioning is off, or the item has just one version. Enable versioning in the list/library settings and guard your flow with `length(...value) >= 2` before comparing.

**Q3: How do I read Person, Lookup or Choice columns from a version?**

Text and Choice columns come back directly. For Person/Lookup, select the related fields in the URI (e.g. `?$select=*,Editor/Title&$expand=Editor`) or read the raw `FieldNameId` value, version entities behave like normal list item fields.

**Q4: Does this work for the file content, not just metadata?**

No. This compares **metadata/column** versions. To compare the binary document contents you'd fetch each version's file stream via `GetFileVersion`/`_vti_history` and diff the documents separately.

## 🔗 Related Tips
- [#PowerPlatformTip 154: Preserve Author, Editor & Dates on Copied SharePoint Files](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-154-preserve-author-created-metadata-power-automate/), keeping history and metadata intact when moving files.
- [#PowerPlatformTip 133: SharePoint Updates with Power Automate, No Required Fields Needed](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-133-sharepoint-updates-with-power-automate-no-required-fields-needed/), writing back only the fields you need.
- [#PowerPlatformTip 58: HTTP Actions](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-58-http-actions/), the basics of *Send an HTTP request to SharePoint*.
