---
title: "#PowerPlatformTip 33: 'Coalesce in Power Apps'"
seo_title: "Coalesce in Power Apps – Return the First Non-Blank Value"
date: 2023-03-07
last_modified_at: 2026-08-22
categories:
  - Article
  - PowerPlatformTip
tags:
  - powerapps
  - power automate
  - coalesce
  - formulas
  - efficiency
excerpt: "Coalesce in Power Apps returns the first non-blank value from a list – replace nested If(IsBlank()) checks with one function. Syntax, examples and the Power Automate equivalent."
description: "How to use Coalesce in Power Apps and Power Automate: syntax, copy-paste examples, Coalesce vs. If(IsBlank()) comparison, and the coalesce() expression for flows."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
faq:
  - question: "What does Coalesce do in Power Apps?"
    answer: "Coalesce evaluates its parameters in order and returns the first value that is not blank and not an empty string. Example: Coalesce(txtNickname.Text, txtFirstName.Text, \"Unknown\")."
  - question: "What is the difference between Coalesce and If(IsBlank())?"
    answer: "Coalesce(a, b) is the short form of If(!IsBlank(a), a, b) – same result, but each value is only written once, which keeps formulas shorter, easier to read and less error-prone with many fallbacks."
  - question: "Does Coalesce exist in Power Automate?"
    answer: "Yes, as the expression coalesce(), for example coalesce(triggerBody()?['email'], variables('fallbackEmail'), 'noreply@example.com'). It returns the first non-null parameter."
  - question: "How many parameters can I pass to the Coalesce function?"
    answer: "Coalesce can handle multiple parameters, but for optimal performance and readability, limit to 5-10 parameters."
  - question: "What's considered a blank value in Coalesce?"
    answer: "Blank values include empty strings, null values, and the Blank() function result. Zero (0) and false are not considered blank."
---

> **TL;DR:** `Coalesce` returns the **first non-blank value** from a list of parameters – the one-line replacement for nested `If(IsBlank())` checks in Power Apps, and available as `coalesce()` in Power Automate.

## 💡 Challenge
In PowerApps and Power Automate, managing multiple potential blank values often results in complex, nested If statements. This complexity hinders readability and efficiency.

## ✅ Solution
Implement the Coalesce function. It scans through a list of parameters and returns the first non-blank value, significantly simplifying formula complexity.

## 🧩 Syntax and Examples

```powerapps
Coalesce( Value1 [, Value2, ... ] )
```

```powerapps
// Returns the nickname, otherwise the first name, otherwise "Unknown"
Coalesce( txtNickname.Text, txtFirstName.Text, "Unknown" )

// Fallback when a SharePoint column is blank
Coalesce( ThisItem.DisplayName, ThisItem.Title )

// Label with a default for empty lookup values
Coalesce( LookUp(Employees, ID = varID).Email, "no-email@company.com" )
```

**Power Automate equivalent** – the `coalesce()` expression works the same way in flows:

```
coalesce(triggerBody()?['email'], variables('fallbackEmail'), 'noreply@example.com')
```

## ⚖️ Coalesce vs. If(IsBlank())

| Approach | Formula | Drawback |
|---|---|---|
| Nested If | `If(!IsBlank(a), a, If(!IsBlank(b), b, c))` | Every value appears twice, hard to read with 3+ fallbacks |
| Coalesce | `Coalesce(a, b, c)` | None – same result, one function call |

Both return the same result. `Coalesce(a, b)` is simply the short form of `If(!IsBlank(a), a, b)` – each value is only written (and evaluated) once.

## 🔧 How It's Done

* Instead of using multiple nested If statements to check each value, list the values as parameters in the Coalesce function.

* The function evaluates each parameter in order and returns the first non-blank value it encounters.

## 🎉 Result
This streamlines your formulas, making them easier to read and maintain, improving both development and future maintenance of the app or flow.

## 🌟 Key Advantages
🔸 Enhanced readability.

🔸 Reduced complexity.

🔸 Easier maintenance.

🔸 Improved performance in certain scenarios.

## 🎥 Video Tutorial
{% include video id="ruRDG-xCbKs" provider="youtube" %}

---

## 🛠️ FAQ
**1. What is the difference between Coalesce and If(IsBlank())?**

`Coalesce(a, b)` is the short form of `If(!IsBlank(a), a, b)` – same result, but each value is only written once. With several fallbacks this keeps formulas dramatically shorter and less error-prone.

**2. Does Coalesce exist in Power Automate?**

Yes, as the expression `coalesce()`, e.g. `coalesce(triggerBody()?['email'], 'noreply@example.com')`. It returns the first non-null parameter.

**3. How many parameters can I pass to the Coalesce function?**

Coalesce can handle multiple parameters, but for optimal performance and readability, limit to 5-10 parameters.

**4. Does Coalesce work with different data types?**

Yes, but ensure all parameters are compatible data types or use conversion functions like Text() or Value() when needed.

**5. What's considered a "blank" value in Coalesce?**

Blank values include empty strings (""), null values, and the Blank() function result. Zero (0) and false are not considered blank.

---
