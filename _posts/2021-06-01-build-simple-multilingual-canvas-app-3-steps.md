---
title: "Build a Simple Multilingual Canvas App in 3 Steps"
date: 2021-06-01
last_modified_at: 2026-09-01
categories:
  - Article
  - PowerPlatformTip
tags:
  - PowerApps
  - Localization
  - Collections
  - SharePoint
  - PowerFx
excerpt: "Make a canvas app multilingual without editing every label. Store translations in one data source, load them into a single collection at OnStart based on Language(), and reference one renamed 'Value' column everywhere. Adding a new language then means just 2 changes."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
---

> **TL;DR:** Keep all translations in one table (one column per language), load the right column into a `colTranslations` collection at `OnStart` using `Left(Language(), 2)`, rename it to a fixed `Value` column, and reference `LookUp(colTranslations, ID = n).Value` in every label. Adding a language = 2 small edits.

Most multilingual Power Apps examples force you to touch every single control when a new language is added. This pattern keeps maintenance tiny: your labels never change, and a new language only needs a new source column plus one line in a `Switch`.

## 💡 Challenge
You want a canvas app that shows text in the user's language (e.g. **en**, **fr**, **de**) and picks it up automatically from the browser/user language. The common approaches scatter `If(Language() = ...)` logic across dozens of controls, so every new language means editing every text property — slow and error-prone.

## ✅ Solution
Centralize translations in **one data source** with a reference column (an ID) and **one column per language**. At startup, detect the language with `Language()`, take the first two letters with `Left(Language(), 2)`, and load only that language column into a collection — renamed to a fixed name (`Value`) so your labels never have to change. A dropdown lets users switch language at runtime.

## 🔧 How it's done

**Step 1 – Create a data source**

🔸 Create a SharePoint list (or Dataverse table / static table) named `MultiLanguage` with one column per language: `en`, `fr`, `de`.

🔸 Use a stable reference column — the auto-generated `ID` works well. Optionally add a `Screen`/`Remark` column to note where each string is used.

| ID | en | fr | de |
|----|----|----|----|
| 1 | Welcome | Bienvenue | Willkommen |
| 2 | Save | Enregistrer | Speichern |

**Step 2 – Define `App.OnStart`**

🔸 Read the user/browser language, keep the first two letters, store it in a variable, and load the matching column into `colTranslations`, renaming it to `Value` with `RenameColumns(ShowColumns(...))`.

```power
Set(
    varLanguage,
    Left( Language(), 2 )
);
Switch(
    varLanguage,
    "fr",
    ClearCollect( colTranslations, RenameColumns( ShowColumns( MultiLanguage, "ID", "fr" ), "fr", "Value" ) ),
    "de",
    ClearCollect( colTranslations, RenameColumns( ShowColumns( MultiLanguage, "ID", "de" ), "de", "Value" ) ),
    // default / fallback = English
    ClearCollect( colTranslations, RenameColumns( ShowColumns( MultiLanguage, "ID", "en" ), "en", "Value" ) )
)
```

🔸 **Tip:** Use the last `Switch` argument as the **default (English) fallback**, so an unsupported language never shows blank text — this matches Microsoft's recommended fallback pattern.

**Step 3 – Insert text elements**

🔸 In each label (or any text property), reference the string by its ID:

```power
LookUp( colTranslations, ID = 1 ).Value
```

Because the column is always called `Value`, you never edit a control when you add a language.

**Bonus – Let users switch language at runtime**

🔸 Add a dropdown with `en`, `fr`, `de`. In its `OnChange`, reuse the Step 2 logic, but set the variable from the selection:

```power
Set( varLanguage, Self.Selected.Value );
Switch(
    varLanguage,
    "fr",
    ClearCollect( colTranslations, RenameColumns( ShowColumns( MultiLanguage, "ID", "fr" ), "fr", "Value" ) ),
    "de",
    ClearCollect( colTranslations, RenameColumns( ShowColumns( MultiLanguage, "ID", "de" ), "de", "Value" ) ),
    ClearCollect( colTranslations, RenameColumns( ShowColumns( MultiLanguage, "ID", "en" ), "en", "Value" ) )
)
```

🔸 You can reduce this to the minimum and put it behind flag images with `OnSelect` instead of a dropdown.

## 🎉 Result
The app shows the right language automatically and users can switch on the fly. Adding a new language (e.g. Italian `it`) takes just **2 changes**: add an `it` column to the data source and add one `"it"` branch to the two `Switch` blocks. No label edits, ever.

## 🌟 Key Advantages
🔸 **Low maintenance:** new language = 2 edits, zero control changes.

🔸 **One reference pattern:** every label uses `LookUp(colTranslations, ID = n).Value`.

🔸 **Automatic detection:** `Left(Language(), 2)` matches your two-letter columns.

🔸 **Safe fallback:** the `Switch` default returns English for unsupported languages.

🔸 **No premium connector:** works with SharePoint, Dataverse or a static in-app table.

## 🛠️ FAQ

**Q1: Why `Left(Language(), 2)`?**

`Language()` returns a full tag like `de-DE` or `en-GB`. Taking the first two letters gives `de` / `en`, which matches the two-letter column names in your data source. If you need region-specific text (e.g. `pt-BR` vs `pt-PT`), key on the full tag instead.

**Q2: Why rename the column to `Value`?**

So every label references the same fixed column name. When you add a language, only `OnStart` changes — the controls stay untouched.

**Q3: How do I handle numbers, dates and currency?**

Use the `Text()` function with a global-aware format (e.g. `Text(Now(), DateTimeFormat.LongDate)`), which formats according to the user's language automatically.

**Q4: Is there a more modern alternative?**

Yes. Microsoft also documents a reusable **translation component** (component libraries) and you can keep the mapping in **Named Formulas** instead of `OnStart` for faster startup. The collection pattern above stays the simplest, connector-free option for small and medium apps.

## 🔗 Related Tips
- [#PowerPlatformTip 16: 'PowerApps: Collections Cookbook'](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-16-powerapps-collections-cookbook/) — more copy-paste collection recipes.
- [How to Load Multiple Data Sources into ONE Collection](https://www.powerplatformtip.com/article/powerplatform/2022/07/20/how-to-load-multiple-data-sources-into-one-collection/) — combine sources into a single collection.
