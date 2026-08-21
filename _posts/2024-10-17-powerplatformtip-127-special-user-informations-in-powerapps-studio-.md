---
title: "#PowerPlatformTip 127: 'Special User-Informations in PowerApps Studio'"
seo_title: "#PowerPlatformTip 127: Special User-Informations in PowerApps Studio"
date: 2024-10-17
last_modified_at: 2026-08-21
categories:
  - Article
  - PowerPlatformTip
tags:
  - PowerApps
  - PowerFx
  - StudioMode
  - UserContext
  - AppSecurity
  - PowerPlatformTip
excerpt: "Automatically switch between test and production user emails in PowerApps Studio using environment detection for secure, automated app testing and deployment."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
---

> **TL;DR:** Auto-switch to a test email in Power Apps Studio and the real `User().Email` in production via `IsBlank(Host.Version)`.

<!-- UPDATE 2026-08-21: (1) Replaced StartsWith(Host.Version, "PowerApps-Studio") with IsBlank(Host.Version) - per Microsoft Learn (Host object reference) Host.Version is always an empty string in Power Apps Studio and only returns a version identifier in the web/mobile player. (2) Fixed the placement: named-formula syntax (name = expr;) cannot be used in App.OnStart. Option A now uses Set() in App.OnStart; Option B uses Named Formulas in App.Formulas. FAQ 1 and FAQ 3 updated; last_modified_at bumped. -->

> **🔄 Update (2026-08-21):** Two corrections. (1) The Studio check now uses `IsBlank(Host.Version)` instead of `StartsWith(Host.Version, "PowerApps-Studio")` - per the [official Microsoft documentation](https://learn.microsoft.com/power-platform/power-fx/reference/object-host), `Host.Version` is **always an empty string in Power Apps Studio** and only returns a value in the web or mobile player. (2) Placement fixed: **named-formula syntax (`name = expr;`) can't go in `App.OnStart`** - use `Set()` in `App.OnStart` (Option A) or define **Named Formulas** in `App.Formulas` (Option B).

## 💡 Challenge
While developing apps in Power Apps Studio, you often want to use test values, like a test email address, without risking that they end up in production.

## ✅ Solution
Check whether the app is running in Studio mode. If it is, use a test email for development; in production the app automatically switches to the real user's email address.

## 🔧 How It's Done

Detect Studio mode with `IsBlank(Host.Version)` and switch the email accordingly. Pick **one** of these placements.

**Option A - in `App.OnStart` (use `Set`):**

```powerfx
Set(fxIsStudioMode, IsBlank(Host.Version));
Set(
    fxUserEmail,
    If(
        fxIsStudioMode,
        "testaccount@company.com",
        User().Email
    )
);
```

**Option B - as Named Formulas in `App.Formulas` (not `OnStart`):**

```powerfx
fxIsStudioMode = IsBlank(Host.Version);
fxUserEmail =
    If(
        fxIsStudioMode,
        "testaccount@company.com",
        User().Email
    );
```

🔸 **Named formulas** live only in `App.Formulas`, are always evaluated automatically and stay up to date - but they **can't** be placed in `App.OnStart`.

🔸 Inside `App.OnStart` use `Set()` for a **global** variable (or `UpdateContext()` for a **screen-scoped** one).

🔸 Either way you get the test email during development and the correct user email in production, with no manual changes.

## 🎉 Result
No more accidentally shipping apps with hardcoded test emails, the switch is seamless and automatic.

## 🌟 Key Advantages

🔸 Prevents accidental deployment of test data

🔸 Saves time by automating email assignment

🔸 Enhances app security and consistency

Special thanks to [Matthew Devaney](https://www.linkedin.com/in/matthew-devaney) for sharing this fantastic Power Apps tip!

---

## 🎥 Video Tutorial
{% include video id="640i6HAngNU" provider="youtube" %}

---

## 🛠️ FAQ
**1. How do I determine if my app is in Studio Mode?**
Check whether `Host.Version` is empty with `IsBlank(Host.Version)`. It returns `true` in Power Apps Studio and `false` in the web or mobile player. (Avoid `StartsWith(Host.Version, "PowerApps-Studio")`, because `Host.Version` is blank in Studio.)

**2. Can I apply this pattern to other test values?**
Yes, use the same logic to switch between development and production values for any parameter.

**3. Where should I place these formulas in my app?**
Use `Set()` in **App.OnStart** (Option A) for a global variable, or define them as **Named Formulas** in **App.Formulas** (Option B). Named-formula syntax (`name = expr;`) can't be placed in `App.OnStart`; for a screen-scoped value use `UpdateContext()`.

## 🔗 Related Tips
- [#PowerPlatformTip 126: User Context for Automated Tasks](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-126-user-context-for-automated-tasks/), run tasks in each user's own context.
- [#PowerPlatformTip 73: Quick Switch to Maker Mode](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-73-quick-switch-to-maker-mode/), speed up your Studio workflow.
