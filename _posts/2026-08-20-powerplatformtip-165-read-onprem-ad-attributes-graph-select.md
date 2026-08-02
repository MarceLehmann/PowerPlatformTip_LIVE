---
title: "#PowerPlatformTip 165: 'Read On-Prem AD Attributes via the User Profile and $select'"
date: 2026-08-13
categories:
  - Article
  - PowerPlatformTip
tags:
  - PowerAutomate
  - EntraID
  - GraphAPI
  - ActiveDirectory
  - Select
  - PowerPlatformTip
excerpt: "onPremisesSamAccountName, onPremisesDistinguishedName, extensionAttribute1-15 and the other on-prem AD fields are hidden by default in Microsoft Graph. Add a $select to a GET /users call and they appear - the synced Active Directory data, straight into your flow."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
---

> **TL;DR:** The on-prem AD attributes synced by Entra Connect (`onPremisesSamAccountName`, `onPremisesDistinguishedName`, `onPremisesExtensionAttributes`, …) are **not** returned by a default user profile lookup. Add `?$select=onPremisesSamAccountName,onPremisesExtensionAttributes,…` to a Graph `GET /users/{upn}` and they show up. Run it via *HTTP with Microsoft Entra ID*.

You need a user's old `samAccountName`, their AD `distinguishedName`, or one of the Exchange custom attributes (`extensionAttribute1-15`) inside a flow - classic hybrid scenarios like matching a line-of-business system or writing an on-prem-style username. You read the user profile... and the fields are empty or missing. They're not missing - Graph just doesn't hand them over unless you explicitly ask.

## 💡 Challenge
Your users are synced from on-premises Active Directory via Microsoft Entra Connect, so their AD attributes live in Entra ID as `onPremises*` properties. But a standard "Get user profile" only returns a small default set (displayName, mail, UPN, job title…). The on-prem fields you actually need - `samAccountName`, `distinguishedName`, the extension attributes - simply aren't in the response.

## ✅ Solution
Microsoft Graph returns only a limited default property set for a user; every `onPremises*` attribute is flagged **"requires `$select` to retrieve"**. So name them explicitly in a `$select` on a `GET /users/{id-or-upn}` call and Graph includes them. The cleanest way from a flow is the preauthorized *HTTP with Microsoft Entra ID* action (see #138) - no app registration.

> ⚠️ **Note:** The standard *Office 365 Users → Get user profile (V2)* action exposes only its own fixed field list and doesn't surface the `onPremises*` attributes - even with its Select box. That's why we go to Graph directly here.

## 🔧 How it's done

**1. Call Graph with an explicit $select.**

🔸 *HTTP with Microsoft Entra ID* → Method `GET`, Uri (URL-encode the `$select` list, or let the action handle it):

```
https://graph.microsoft.com/v1.0/users/jane@contoso.com?$select=displayName,onPremisesSamAccountName,onPremisesUserPrincipalName,onPremisesDomainName,onPremisesDistinguishedName,onPremisesSecurityIdentifier,onPremisesImmutableId,onPremisesSyncEnabled,onPremisesLastSyncDateTime,onPremisesExtensionAttributes
```

🔸 Without the `$select`, none of these come back. With it, the JSON response contains each requested field.

**2. Read the values from the response.**

🔸 The synced AD login name is `body('HTTP')?['onPremisesSamAccountName']`, the OU path is `body('HTTP')?['onPremisesDistinguishedName']`, and the Exchange custom attributes sit under `onPremisesExtensionAttributes`, e.g. `body('HTTP')?['onPremisesExtensionAttributes']?['extensionAttribute1']`.

**3. Confirm the user is actually synced.**

🔸 Check `onPremisesSyncEnabled`: if it's `true`, the `onPremises*` values are authoritative (read-only, sourced from AD). For cloud-only users it's `null`/`false` and the on-prem fields are empty - handle that branch so your flow doesn't break on cloud accounts.

**4. Reuse the value.**

🔸 Feed `onPremisesSamAccountName` into an on-prem SQL/LOB lookup, build a `DOMAIN\samAccountName` string, or map `extensionAttribute*` to your own business logic - all inside the same flow.

## 🎉 Result
Your flow now reads the real on-premises AD attributes - `samAccountName`, `distinguishedName`, the extension attributes and more - straight from the synced Entra ID user object. No AD connector, no on-prem gateway, just one Graph `GET` with the right `$select`.

## 🌟 Key Advantages

🔸 **The hidden fields, revealed:** `$select` is the only switch that turns on the `onPremises*` properties.

🔸 **No gateway, no AD connector:** you read the AD data that Entra Connect already synced into the cloud.

🔸 **No app registration:** *HTTP with Microsoft Entra ID* authenticates with the connection's identity (#138).

🔸 **Hybrid-friendly:** perfect for matching users to legacy systems that still key on `samAccountName` or an OU.

## 🛠️ FAQ

**Q1: Why are the fields empty even with `$select`?**

The `onPremises*` attributes are only populated for users synchronized from on-premises AD via Microsoft Entra Connect. For cloud-only accounts they're empty by design - check `onPremisesSyncEnabled` first.

**Q2: How do I get the Exchange custom attributes 1-15?**

Select `onPremisesExtensionAttributes` (the whole complex object), then drill in: `...?['onPremisesExtensionAttributes']?['extensionAttribute3']`. Each holds up to 1024 characters and is read-only for synced users.

**Q3: Which Graph permission do I need?**

Reading these is a standard user read: `User.Read.All` (delegated or application) is sufficient. No special directory-write scope is required just to read the profile.

**Q4: Can't I just use the Office 365 Users connector?**

Its *Get user profile (V2)* action returns a fixed schema that doesn't include the `onPremises*` attributes, so its Select box won't surface them. Go straight to Graph `GET /users` with `$select` to get these fields reliably.

## 🔗 Related Tips
- [#PowerPlatformTip 138: Graph API via HTTP with Microsoft Entra ID](https://www.powerplatformtip.com/article/powerplatformtip/PowerPlatformTip-138-Graph-API-HTTP/), the auth-free way to run this GET.
- [#PowerPlatformTip 95: Optimized SharePoint Queries](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-95-optimized-sharepoint-queries/), more on getting the most out of `$select` and `$filter`.
- [#PowerPlatformTip 58: HTTP Actions](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-58-http-actions/), the fundamentals of calling REST APIs from a flow.
