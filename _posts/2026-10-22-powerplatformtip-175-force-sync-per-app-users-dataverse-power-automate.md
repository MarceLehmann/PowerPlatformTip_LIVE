---
title: "#PowerPlatformTip 175: 'Force-Sync Per-App-Licensed Users into Dataverse with Power Automate'"
date: 2026-10-22
categories:
  - Article
  - PowerPlatformTip
tags:
  - PowerAutomate
  - Dataverse
  - ManagedEnvironments
  - Licensing
  - Governance
  - PowerPlatformTip
excerpt: "In managed production environments with per-app (app-pass) licensing, users are NOT auto-synced into the Dataverse SystemUser table – they only appear on demand. Automate it: trigger a flow when a user joins the environment's Entra security group and call the 'Force Sync User' admin action so access is ready before the first login."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
---

> **TL;DR:** With per-app (app-pass) licensing, the background sync does **not** add users to the Dataverse `SystemUser` table automatically – they are only added *on demand*. Trigger a flow when a user is added to the environment's Microsoft Entra security group and call **Force Sync User** (Power Platform for Admins connector) so the user record exists *before* they first open the app.

Managed production environments licensed with **per-app plans (app-pass)** behave differently from Dynamics 365 / Power Apps Premium seats. A licensed Premium user is picked up by the periodic background sync and lands in Dataverse automatically. An **app-pass** user is **not** – Microsoft only adds them *on demand*: at their first sign-in (just-in-time), via an admin action, or via the API. That gap causes "You are not a member of the organization" errors and missing security-role assignments right when a new user tries to start. This tip automates the fill so the `SystemUser` record is provisioned in advance.

## 💡 Challenge
You run a **managed production environment** and license makers/users with **per-app (app-pass)** plans, scoped by a Microsoft Entra security group on the environment.
Per Microsoft's documented rules, users in *"an environment with a Dataverse database and environment-level app-pass license type"* are **not added automatically** to the Dataverse `SystemUser` table – they are added on demand only.
The symptoms: a freshly added user isn't visible under **Users + permissions**, has no security role, and hits access errors on first launch. Manually clicking **Refresh** in the Power Platform admin center for every new joiner doesn't scale.

## ✅ Solution
Automate the *on-demand* provisioning. Microsoft exposes a **Force Sync User** action in the **Power Platform for Admins** connector that triggers the same server-side sync the admin center's *Refresh* uses. Combine it with a trigger that fires **when a user is added to the environment's associated Entra security group**, and the Dataverse `SystemUser` record is created before the user ever signs in.
Use an **Environment Group** to consistently govern the managed production environments you want this to cover (grouped policies, security-group requirements, sharing limits), so your automation targets a well-defined, governed set of environments.

## 🔧 How it's done

**1. Confirm the license/sync gap.**

🔸 In the Power Platform admin center, open the environment → **Settings → Users + permissions → Users**. If a licensed user is missing right after assignment, you are in the app-pass on-demand scenario (not a bug).

**2. Govern the target environments with an Environment Group.**

🔸 Admin center → **Manage → Environment groups** → create/assign a group for your managed production environments. Enforce **"environment must be associated with a security group"** so membership is the single source of truth for who belongs.

**3. Pick the trigger: "user added to the environment's Entra security group".**

🔸 Use **When a user is added to the group** (Microsoft Entra ID / Azure AD connector) on the security group that is associated with the environment.

🔸 Capture the new member's **object ID** (`userId` / `id`) – that's what the sync action needs, not the UPN alone.

**4. Call the Force Sync User admin action.**

🔸 Add **Power Platform for Admins → Force Sync User**.

🔸 Map the parameters:

```
Environment  : <your managed production environment name/id>
User Id      : <object id of the added user>
```

🔸 For multiple governed environments, wrap this in an **Apply to each** over your environment list so one new joiner is synced into every relevant environment.

**5. (Optional but recommended) Assign the security role right after.**

🔸 Because syncing the user does **not** grant privileges, follow up with a Dataverse **Associate** (systemuserroles) or the admin API to attach the correct security role. Provisioning without a role still yields a "no roles" screen.

**6. Handle timing and idempotency.**

🔸 The sync is not always instant. Add a short **Delay** or a retry (Do until the user appears via *List records* on the `systemusers` table) before assigning roles.

🔸 The action is safe to re-run – if the user already exists, it refreshes rather than duplicates.

## 🎉 Result
New per-app-licensed users are provisioned into the Dataverse `SystemUser` table **proactively**, the moment they join the environment's security group – no manual *Refresh*, no first-login errors, and roles can be assigned immediately after. Onboarding into managed production environments becomes a hands-off, governed process.

## 🌟 Key Advantages

🔸 **Fixes the app-pass blind spot:** targets exactly the license type Microsoft excludes from automatic sync.

🔸 **Proactive, not reactive:** users are ready *before* their first login instead of erroring on it.

🔸 **Governed at scale:** Environment Groups + a single associated security group keep membership authoritative across many environments.

🔸 **Uses documented admin tooling:** the same *Force Sync User* mechanism as the admin center's Refresh, just automated.

## 🎥 Video Tutorial
{% include video id="XXXXXXXXXXX" provider="youtube" %}

## 🛠️ FAQ

**Q1: Why aren't per-app-licensed users synced automatically like Premium users?**

Microsoft's documented rule: users in *an environment with a Dataverse database and environment-level app-pass license type* are added to Dataverse **on demand only** (first access, API, or admin center), not by the periodic background sync. Dataverse-for-Teams and free-Dataverse-plan (Office) users fall in the same on-demand bucket.

**Q2: What exactly does "Force Sync User" do?**

It triggers the server-side, on-demand user sync for a single user in a specific environment – the same operation as clicking **Refresh** on a user in the admin center. It creates/updates the `SystemUser` record; it does **not** assign security roles.

**Q3: Does syncing give the user access to apps and data?**

No. Adding the `SystemUser` record only makes the user *present* in the environment. You still must assign the appropriate **security role** (directly or via a team) for actual privileges.

**Q4: Which trigger should fire the flow?**

Trigger on membership of the **Microsoft Entra security group that is associated with the environment** ("When a user is added to the group"). Since a valid app-pass user must be in that group anyway, group membership is the natural, authoritative signal.

**Q5: Can I cover several environments at once?**

Yes. Keep your managed production environments in an **Environment Group** and loop the *Force Sync User* action over that environment list, so one new joiner is provisioned everywhere they belong.

## 🔗 Related Tips
- [#PowerPlatformTip 87: Licensing, Sharing Canvas Apps with Guests](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-87-licensing-sharing-canvas-apps-with-guests/), the licensing context for who can be synced at all.
- [#PowerPlatformTip 74: Elevated Permissions](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-74-elevated-permissions/), impersonation via flow – which itself triggers a just-in-time user sync.
- [#PowerPlatformTip 135: One Flow, Many Users](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-135-one-flow-many-users/), patterns for user-context automation across many people.
