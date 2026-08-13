---
title: "#PowerPlatformTip 176: 'Get a Free Power Platform Developer Plan with a Personal Email'"
date: 2026-10-29
categories:
  - Article
  - PowerPlatformTip
tags:
  - PowerPlatform
  - DeveloperPlan
  - Dataverse
  - Entra
  - Azure
  - Environments
  - PowerPlatformTip
excerpt: "No work or school email? Spin up your own Microsoft Entra tenant from a personal email, create a member user, then claim the free Power Apps Developer Plan with that user to get your own Dataverse developer environments."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
---

> **TL;DR:** No work or university email, no problem. Create your own Microsoft Entra tenant from a personal email via an Azure sign-up, add a real member (work) user, then claim the free **Power Apps Developer Plan** with that user to get your own Dataverse-enabled developer environments.

The **Power Apps Developer Plan** is free and gives you your own Dataverse-enabled developer environments to build, test and share Power Apps and Power Automate solutions. The catch: sign-up needs a **work or school account**. A personal Microsoft account (outlook.com, gmail, …) is rejected. The fix is to create your own Microsoft Entra ID tenant and a proper member user inside it, then claim the plan with that user.

## 💡 Challenge
You want a cost-free, Dataverse-backed space to learn and prototype, but you only have a personal email. The Developer Plan sign-up asks for a **work or school account** and won't accept your personal Microsoft account, so you're stuck before you even start.

## ✅ Solution
Create your own tenant. Signing up for an Azure subscription with a personal email provisions a **Microsoft Entra ID** tenant where you are the account administrator. Inside that tenant you create a real **member user** (`you@yourtenant.onmicrosoft.com`) — that is a proper organizational identity the Developer Plan accepts. Claim the plan with that new user, not with the personal account you signed up with.

## 🔧 How it's done

**1. Create a tenant from your personal email.**

🔸 Go to [azure.microsoft.com](https://azure.microsoft.com/) and start a subscription (Pay-as-you-go works). A credit card is required **on file**, but the Developer Plan itself costs nothing — you're only billed if you consume paid Azure services.

🔸 This provisions a **Microsoft Entra ID** tenant with a default `*.onmicrosoft.com` domain, with you as the account admin.

**2. Create a work/school user in Microsoft Entra.**

🔸 Open the **Microsoft Entra admin center** → **Users** → **All users** → **New user** → **Create new user**.

🔸 Give it a name and a user principal name like `dev@yourtenant.onmicrosoft.com`. This member account is the organizational identity the Developer Plan will accept.

🔸 Optionally assign the **Global Administrator** role so the account can fully manage the tenant, users and environment settings.

**3. Claim the Developer Plan with the new user.**

🔸 Sign in as the **new** user (not the personal account) and go to [aka.ms/PowerAppsDevPlan](https://aka.ms/PowerAppsDevPlan) (or the [Power Apps Developer Plan page](https://www.microsoft.com/power-platform/products/power-apps/free)).

🔸 Activate the plan. You now get your own developer environment(s) with Dataverse.

**4. Verify with a Dataverse table.**

🔸 Open [make.powerapps.com](https://make.powerapps.com/), select your developer environment, and create a **table** in Dataverse.

🔸 If the table provisions successfully, Dataverse is live and your environment is set up correctly.

**Beating the 3-environment ceiling — it's per maker, not per tenant.** Each **maker can have up to three developer environments**, and they **don't count against your tenant's storage capacity**. Need more? Create additional member users in your Entra tenant and claim the Developer Plan for each one — every maker gets their own three. Keep in mind that developer environments unused for **90 days** are automatically turned off and later removed if you don't respond to the notifications, and **Dynamics 365 apps aren't available** in developer environments.

## 🎉 Result
Starting from nothing but a personal email, you now own a Microsoft Entra tenant, a work user, and free Dataverse-enabled developer environments — a proper, isolated workspace to build and test Power Apps and Power Automate solutions at no cost.

## 🌟 Key Advantages

🔸 **Zero license cost:** the Power Apps Developer Plan is free; the card on file is only for Azure consumption.

🔸 **Full Dataverse:** real tables, relationships and solutions for realistic prototyping — not just SharePoint lists.

🔸 **Scales past three:** the limit is per maker, so extra Entra users unlock more developer environments.

🔸 **No capacity hit:** developer environments don't consume tenant storage capacity.

## 🎥 Video Tutorial
{% include video id="VIDEO_ID" provider="youtube" %}

## 🛠️ FAQ

**Q1: Will my credit card be charged?**

Not for the Developer Plan — it's free. The card sits on file for the Azure subscription and is only billed if you actually consume paid Azure services.

**Q2: Why doesn't my personal Outlook account work directly?**

The Developer Plan requires a **work or school (Entra) account**. A personal Microsoft account isn't an organizational identity, so you create a member user in your own tenant and claim the plan with that.

**Q3: How many developer environments do I get?**

Up to **three per maker**. They don't count against tenant capacity. To get more, create additional Entra users and claim the Developer Plan for each.

**Q4: Do I really need Global Administrator on the new user?**

Not strictly for the plan itself, but it lets that account manage the tenant, users and environment settings — convenient when you're the only admin.

**Q5: Do developer environments expire?**

They aren't time-limited while in use, but any developer environment **unused for 90 days** is automatically turned off and later removed if you ignore the notifications. Dynamics 365 apps also aren't available in them.

## 🔗 Related Tips
- [#PowerPlatformTip 90: Power Platform Community Developer Plan](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-90-power-platform-community-developer-plan/), what the free Developer Plan gives you.
- [#PowerPlatformTip 164: Add External Users to Microsoft Entra ID with Power Automate](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-164-invite-external-guest-users-entra-id-power-automate/), automate identity management in your tenant.
- [#PowerPlatformTip 145: Power Platform Tools](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-145-power-platform-tools/), a browser toolbox to speed up development once your environment is live.

---
*Special thanks to Josh Killity at Platform Pulse Solutions, whose written walkthrough covers the same setup plus custom domains and going beyond the three-environment limit.*
