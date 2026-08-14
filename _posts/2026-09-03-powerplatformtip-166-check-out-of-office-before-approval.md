---
title: "#PowerPlatformTip 166: 'Check if a Person Is Out of Office Before an Approval'"
date: 2026-09-17
categories:
  - Article
  - PowerPlatformTip
tags:
  - PowerAutomate
  - Approvals
  - Outlook
  - MailTips
  - GraphAPI
  - PowerPlatformTip
excerpt: "Approvals stall when the approver is on holiday. Use the standard Office 365 Outlook 'Get mail tips for a mailbox (V2)' action to detect automatic replies before you assign - then reroute to a deputy. Need the exact status? Fall back to Graph mailboxSettings."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
---

> **TL;DR:** Use the standard **Office 365 Outlook → "Get mail tips for a mailbox (V2)"** action with the `automaticReplies` option to see whether an approver is away - no HTTP, no app registration. Need the exact on/scheduled *status* or a future window? Fall back to Graph `GET /users/{upn}/mailboxSettings/automaticRepliesSetting`.

An approval lands in the inbox of someone who left for a two-week holiday this morning. The flow waits. The requester is blocked. Nobody notices until the deadline is gone. The information you needed was there all along - the approver had an out-of-office reply switched on - you just never asked before routing the task.

## 💡 Challenge
Approvals silently rot in the mailbox of an absent approver. A "Start and wait for an approval" can wait up to 30 days (see #144), so a single person on leave can freeze an entire business process - and you only find out when it's already too late to reroute.

## ✅ Solution
Before you assign the approval, check whether the approver has automatic replies turned on. The simplest way needs **no HTTP action and no app registration**: the standard *Office 365 Outlook* connector ships a **Get mail tips for a mailbox (V2)** action that returns the recipient's automatic-reply MailTip - including the reply text. If you need the precise on/off/scheduled *status* (or a reply that is scheduled but hasn't started yet), fall back to Microsoft Graph `mailboxSettings`.

## 🔧 How it's done

**1. Read the mail tip (standard, no-code).**

🔸 *Office 365 Outlook* → **Get mail tips for a mailbox (V2)**. Set the mailbox to the approver's address and request the `automaticReplies` option. The connector runs under its own signed-in connection - no app registration, no `HTTP` action.

**2. Decide whether the approver is away.**

🔸 Check the returned automatic-reply message, e.g. `body('Get_mail_tips_for_a_mailbox_(V2)')?['automaticReplies']?['message']`. A non-empty value means out-of-office is active - and you even get the reply text to quote in your escalation note.

**3. (Optional) Get the exact status.**

🔸 If you need to distinguish `disabled` vs `alwaysEnabled` vs `scheduled`, or spot a reply that is scheduled for the *future*, call Graph via *HTTP with Microsoft Entra ID* (#138): `GET https://graph.microsoft.com/v1.0/users/jane@contoso.com/mailboxSettings/automaticRepliesSetting`. Read `body('HTTP')?['status']` plus `scheduledStartDateTime`/`scheduledEndDateTime` (each carries a `dateTime` and a `timeZone`), and compare against `utcNow()` to see whether the window is active right now.

**4. Branch the approval.**

🔸 If the approver is away, point *Start and wait for an approval* at a deputy or their manager (`GET /users/{upn}/manager`) - otherwise assign as usual. Optionally add the out-of-office text to the approval details so the backup approver has the context.

## 🎉 Result
Your approval flow now checks availability before it assigns anything. Absent approvers are detected up front, the task is rerouted to a deputy or manager automatically, and no request sits frozen in an empty mailbox for weeks.

## 🌟 Key Advantages

🔸 **Zero setup:** the Office 365 Outlook "Get mail tips" action needs no app registration and no premium HTTP action.

🔸 **Reusable context:** the automatic-reply text comes back with the tip, ready to drop into your escalation message.

🔸 **Optional precision:** the Graph `mailboxSettings` fallback adds the exact `status` and reveals replies scheduled for the future.

🔸 **No more frozen approvals:** availability is checked before the task is ever assigned.

## 🛠️ FAQ

**Q1: Which action exactly?**

Office 365 Outlook → **Get mail tips for a mailbox (V2)**. It surfaces MailTips such as automatic replies / out-of-office and mailbox-full status. Note it isn't available in GCC High or Mooncake environments.

**Q2: Which permission do I need?**

The connector action runs under the signed-in Office 365 Outlook connection (the underlying Graph `getMailTips` uses `Mail.Read`). The Graph `mailboxSettings` fallback needs `MailboxSettings.Read`.

**Q3: MailTip or `mailboxSettings` - what's the difference?**

The MailTip only returns a message when the automatic reply is *active now*. Graph `mailboxSettings/automaticRepliesSetting` also exposes a reply that is `scheduled` but hasn't started yet, plus the explicit `status` and both the internal and external reply texts.

**Q4: The message is empty but the person is away - why?**

External-only replies or remote-domain settings can suppress the automatic-reply MailTip. When in doubt, use the Graph `mailboxSettings` fallback to read the definitive `status`.

## 🔗 Related Tips
- [#PowerPlatformTip 144: Infinite Approvals](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-144-infinite-approvals/), why a waiting approval can hang around for weeks.
- [#PowerPlatformTip 138: Graph API via HTTP with Microsoft Entra ID](https://www.powerplatformtip.com/article/powerplatformtip/PowerPlatformTip-138-Graph-API-HTTP/), the auth-free way to run the `mailboxSettings` fallback.
- [#PowerPlatformTip 126: User Context for Automated Tasks](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-126-user-context-for-automated-tasks/), the flip side - setting out-of-office replies centrally.
