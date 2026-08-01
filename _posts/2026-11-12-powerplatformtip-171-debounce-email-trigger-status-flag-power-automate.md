---
title: "#PowerPlatformTip 171: 'Debounce the Email Trigger with a Status Flag & Modified-Date Window'"
date: 2026-11-12
categories:
  - Article
  - PowerPlatformTip
tags:
  - PowerAutomate
  - SharePoint
  - Outlook
  - Performance
  - PowerPlatformTip
excerpt: "The 'When a new email arrives (V3)' trigger fires many parallel runs during a mail burst. Use a SharePoint status flag: the first run reads the item's Modified date, flips the flag to Yes, then fetches every mail from that Modified date until now. Runs that see Yes do nothing."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
---

> **TL;DR:** Let the flow run in parallel, but guard it with a single SharePoint control row. Each run checks a Yes/No `IsRunning` flag. If it's *Yes*, do nothing. If it's *No*, capture the item's **Modified** timestamp **first**, set the flag to *Yes*, then *Get emails (V3)* + *Filter array* to collect every mail from that Modified date until now, process the batch, and set the flag back to *No*.

When mails arrive in bursts, the **When a new email arrives (V3)** trigger spins up many parallel runs. Instead of throttling them, you let the **first** run become the collector: it grabs the time window from the control item's last **Modified** date, processes everything since then, and every other run simply exits because the flag is already *Yes*.

## 💡 Challenge
On a busy inbox, a short burst of mails makes the trigger start many flow instances at once. That means duplicated work, connector throttling, inconsistent state, and sometimes runs that look skipped. You want the mails processed **once**, as a batch, without missing any that arrived while a run was already busy.

## ✅ Solution
Keep a **status flag** in a tiny SharePoint control list (one row, a Yes/No column `IsRunning`). The flow runs in parallel as normal, but every run first checks the flag:

- **Flag = Yes** → another run already owns the batch → **do nothing**.
- **Flag = No** → this run is the collector. It reads the row's **Modified** date (the moment the flag last changed), flips the flag to *Yes*, then fetches **all** mails received from that Modified date until now and processes them. Finally it resets the flag to *No*.

The `Modified` date is the key: it marks exactly when the previous cycle ended, so the window covers every mail that arrived since — including the ones that fired the parallel runs you're skipping.

## 🔧 How it's done

**1. Create the control list.**

🔸 A SharePoint list `FlowLocks` with a single item, e.g. `Title = EmailProcessor` and a **Yes/No** column `IsRunning` (default *No*). Note its **item ID**.

**2. Leave the trigger parallel.**

🔸 Keep **When a new email arrives (V3)** as is — no concurrency change. Parallelism is fine; the flag does the gating.

**3. Read the control item — capture Modified BEFORE anything else.**

🔸 *Get item* on `FlowLocks` (your control row ID).
🔸 Immediately store its Modified date in a variable `WindowStart`:

```
body('Get_item')?['Modified']
```

⚠️ Read `Modified` **before** you update the item — the update will overwrite it with the current time.

**4. Condition: is it already running?**

🔸 *Condition*: `IsRunning` **is equal to** `true`.

**5. If Yes → do nothing.**

🔸 **If yes** branch → **Terminate** with status *Succeeded*. Another run owns this batch.

**6. If No → claim it and set the flag.**

🔸 **If no** branch → *Update item* on `FlowLocks` → `IsRunning = Yes`.
🔸 This is what bumps `Modified` to now — which is why you captured the old value in step 3.

**7. Collect every mail in the window.**

🔸 *Get emails (V3)* — **Fetch Only Unread = Yes**, raise **Top** (e.g. 25–50) to cover the burst. Results include a `receivedDateTime` property.
🔸 *Filter array* on the returned value, keep only mails since the window start:

```
@greaterOrEquals(item()?['receivedDateTime'], variables('WindowStart'))
```

**8. Process the batch.**

🔸 *Apply to each* over the filtered array → your work (save, notify, create items…).
🔸 *Mark as read or unread (V3)* on each `MessageId` so it isn't picked up again.

**9. Release the flag — always.**

🔸 *Update item* on `FlowLocks` → `IsRunning = No`.
🔸 Open **Configure run after** on this action and also run it after *has failed* / *has timed out*, so an error never leaves the flag stuck on *Yes*.

## 🎉 Result
One collector run absorbs the whole burst: it processes every mail from the previous `Modified` timestamp to now, while the parallel duplicate runs exit instantly. No stampede, less throttling, and nothing slips through the gap between "already running" and "just arrived".

## 🌟 Key Advantages

🔸 **Runs in parallel, still safe:** the flag gates the work, no irreversible concurrency setting needed.

🔸 **No gaps:** the `Modified` date guarantees the window starts exactly where the last cycle ended.

🔸 **Standard connectors only:** Office 365 Outlook + SharePoint, no premium.

🔸 **Self-healing flag:** *Configure run after* resets it even on failure.

## 🛠️ FAQ

**Q1: Why read the `Modified` date before updating the flag?**

Updating the item to `IsRunning = Yes` sets `Modified` to now. If you read it afterwards your window would start "now" and you'd miss the burst. Capture it first, then update.

**Q2: With parallel runs, can two runs both read `No` at the same time?**

There's a tiny race window. In practice the first *Update item* wins and the second batch just re-scans the same window harmlessly (mails already marked read are skipped). If you need strict exclusion, add a follow-up *Get item* check after setting the flag, or fall back to trigger Concurrency = 1.

**Q3: What if a run fails after setting the flag to Yes?**

Use **Configure run after** on the final *Update item* so it also runs on *failed*/*timed out*. That resets `IsRunning` to *No* and the next mail starts a clean cycle.

**Q4: Get emails (V3) has no "received after" filter — how does the window work?**

Right, there's no native date parameter. The action returns `receivedDateTime` per mail, so a *Filter array* with `greaterOrEquals(..., WindowStart)` does the time-window filtering client-side.

**Q5: Can I store the flag in Dataverse instead?**

Yes — any store with a single control record and a "modified" timestamp works. SharePoint is the lightest zero-cost option.

## 🔗 Related Tips
- [#PowerPlatformTip 35: Concurrent Control](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-35-concurrent-control/), the stricter alternative: run instances sequentially with parallelism 1.
- [#PowerPlatformTip 11: Trigger Condition](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-11-trigger-condition/), stop a flow unless criteria are met.
- [#PowerPlatformTip 95: Optimized SharePoint Queries](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-95-optimized-sharepoint-queries/), filter data at the source with OData in Get items.
