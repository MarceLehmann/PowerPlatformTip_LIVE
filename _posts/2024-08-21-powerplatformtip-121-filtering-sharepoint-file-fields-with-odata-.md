---
title: "#PowerPlatformTip 121: 'Filtering SharePoint File Fields with OData'"
seo_title: "Filter SharePoint Files by Name in Power Automate"
date: 2024-08-21
last_modified_at: 2026-08-22
categories:
  - Article
  - PowerPlatformTip
tags:
  - PowerAutomate
  - SharePoint
  - OData
  - FileLeafRef
  - FileRef
  - FileDirRef
  - PowerPlatformTip
excerpt: "Quickly and efficiently filter SharePoint files in Power Automate by using FileLeafRef (file name), FileRef (full path), and FileDirRef (folder) fields with simple OData filter expressions, even if you don't know how these fields work yet."
description: "Filter SharePoint files in Power Automate by name, path, or folder using FileLeafRef, FileRef & FileDirRef in one OData query. No loops, no premium connector."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
faq:
  - question: "How do I exclude files by extension, e.g. ZIP files, with an OData filter?"
    answer: "Use substringof with eq false in the Filter Query: substringof('.zip', FileLeafRef) eq false returns every file whose name does not contain .zip. Combine with eq true to include only certain extensions."
  - question: "How do I apply these OData filters in Power Automate?"
    answer: "In the 'Get files (properties only)' action, enter your OData filter expression into the Filter Query field to limit results based on FileLeafRef, FileRef, or FileDirRef."
  - question: "Can I combine multiple OData filter conditions?"
    answer: "Yes. Use logical operators like and or or in your filter query, for example: FileLeafRef eq 'document.pdf' and startswith(FileDirRef, '/sites/demo/Shared Documents')."
  - question: "Are these fields supported in SharePoint on-premises?"
    answer: "OData filtering with FileLeafRef, FileRef, and FileDirRef works in SharePoint Online and on-premises (2013+). Ensure your environment and connector version support OData queries."
  - question: "What does FileLeafRef mean?"
    answer: "It's the file name itself, for example, 'Invoice.pdf'."
  - question: "What does FileRef mean?"
    answer: "It's the full path of a file, including folders, for example, '/sites/demo/Shared Documents/Invoice.pdf'."
  - question: "What does FileDirRef mean?"
    answer: "It refers to a folder path only, for example, '/sites/demo/Shared Documents/'."
---

> **TL;DR:** Filter SharePoint files in Power Automate with the FileLeafRef, FileRef and FileDirRef fields via simple OData queries.

## 💡 Challenge
Filtering SharePoint documents in Power Automate gets tricky when all you have is a file name or folder path. The right filter fields aren't obvious, so many makers end up fetching every file and filtering inside a loop, slow and inefficient.

## ✅ Solution
Use SharePoint's built-in file-reference fields, `FileLeafRef`, `FileRef` and `FileDirRef`, directly in your OData **Filter Query** to return exactly the files you need, server-side.

## 🔧 How It's Done

Add one of these fields to the **Filter Query** of your "Get files (properties only)" action:

🔸 `FileLeafRef`, filter by file name (without the path), e.g. `FileLeafRef eq 'Invoice.pdf'`.

🔸 `FileRef`, filter by the full path to a file.

🔸 `FileDirRef`, filter by files within a specific folder.

This pairs well with efficient query design, see #PowerPlatformTip 95, 'Optimized SharePoint Queries'.

## 📦 Exclude or Include Files by Extension with substringof

A very common requirement: return all files **except** a certain type. Use `substringof()` on `FileLeafRef` with `eq false`:

```
substringof('.zip', FileLeafRef) eq false
```

This returns every file whose name does **not** contain `.zip`. The reverse works too – only Word documents:

```
substringof('.docx', FileLeafRef) eq true
```

And both patterns can be combined with other conditions:

```
substringof('.zip', FileLeafRef) eq false and startswith(FileDirRef, '/sites/demo/Shared Documents')
```

**Note:** `substringof('.zip', FileLeafRef)` matches anywhere in the file name – a file called `archive.zip.backup.docx` would also match. For a strict extension check, sort by name or verify the extension afterwards with an expression like `endsWith()` in a condition.

## 🎉 Result
Your workflows become more robust and easier to maintain. Server-side filtering cuts the time spent on manual adjustments and minimizes the potential for errors.

## 🌟 Key Advantages

🔸 **Improved efficiency:** faster development through server-side filtering.

🔸 **Better consistency:** a standardized approach across all projects.

🔸 **Enhanced reliability:** reduced risk of failure during execution.

---

## 🎥 Video Tutorial
{% include video id="2DmwfItne0Q" provider="youtube" %}

---

## 🛠️ FAQ
**1. How do I exclude ZIP files (or any extension) with an OData filter?**
Use `substringof('.zip', FileLeafRef) eq false` in the **Filter Query** – it returns every file whose name does not contain `.zip`. With `eq true` you include only matching files instead.

**2. How do I apply these OData filters in Power Automate?**
In the "Get files (properties only)" action, enter your OData filter expression into the **Filter Query** field to limit results based on FileLeafRef, FileRef, or FileDirRef.

**3. Can I combine multiple OData filter conditions?**
Yes. Use logical operators like `and` or `or` in your filter query, for example:
`FileLeafRef eq 'document.pdf' and startswith(FileDirRef, '/sites/demo/Shared Documents')`.

**4. Are these fields supported in SharePoint on-premises?**
OData filtering with FileLeafRef, FileRef, and FileDirRef works in SharePoint Online and on-premises (2013+). Ensure your environment and connector version support OData queries.

**5. What does FileLeafRef mean?**
It's the file name itself, for example, 'Invoice.pdf'.

**6. What does FileRef mean?**
It's the full path of a file, including folders, for example, '/sites/demo/Shared Documents/Invoice.pdf'.

**7. What does FileDirRef mean?**
It refers to a folder path only, for example, '/sites/demo/Shared Documents/'.

## 🔗 Related Tips
- [#PowerPlatformTip 95: Optimized SharePoint Queries](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-95-optimized-sharepoint-queries/), design efficient, delegable OData queries.
- [#PowerPlatformTip 153: Rename a SharePoint File with Power Automate](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-153-rename-sharepoint-file-power-automate/), work with FileLeafRef to rename files in place.
