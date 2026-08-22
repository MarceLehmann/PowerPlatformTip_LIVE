---
title: "#PowerPlatformTip 24: 'Merge Arrays or Tables in Power Automate'"
seo_title: "Merge or Combine Two Arrays in Power Automate (Keep Duplicates)"
date: 2023-01-31
last_modified_at: 2026-08-22
categories:
  - Article
  - PowerPlatformTip
tags:
  - power automate
  - arrays
  - tables
  - data manipulation
  - concat
excerpt: "How to merge or combine two arrays in Power Automate: union() for unique values, or the join-concat-split pattern to keep duplicates. Copy-paste expressions included."
description: "Merge, combine or join two arrays in Power Automate: union() removes duplicates, the join/concat/split pattern keeps them. Both expressions ready to copy."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
faq:
  - question: "How do I combine two arrays in Power Automate?"
    answer: "Use union(array1, array2) if duplicates may be removed, or join each array with a delimiter, concat both strings and split the result back into an array if you need to keep every element including duplicates."
  - question: "How do I merge two arrays without removing duplicates?"
    answer: "union() always removes duplicates. To keep them, use: split(concat(join(array1, '||'), '||', join(array2, '||')), '||')."
  - question: "What happens if my array contains values that include the delimiter '||'?"
    answer: "Choose a unique delimiter that doesn't appear in your data, or use a more complex delimiter like '###DELIMITER###'."
  - question: "Can this method handle arrays with different data types?"
    answer: "Yes, but ensure all values can be converted to strings for the join operation, then convert back to appropriate types if needed."
  - question: "Is there a performance impact when merging large arrays?"
    answer: "For very large arrays, consider using the union() function for simple merging without duplicates, or batch processing for better performance."
---

> **TL;DR:** Merge two arrays and keep duplicates by joining each with a delimiter, concatenating, then splitting back, instead of union which drops duplicates.

## 💡 Challenge
You need to merge two arrays in Power Automate but keep the duplicates. The `union` function merges arrays, but it removes duplicates and returns only unique values.

## 🔀 Combine Two Arrays: union() vs. join/concat/split

There are two ways to combine (merge, join) two arrays in Power Automate – pick the one that matches your duplicate requirement:

| Method | Expression | Duplicates |
|---|---|---|
| **union()** | `union(outputs('array1'), outputs('array2'))` | ❌ removed |
| **join → concat → split** | `split(concat(join(array1, '||'), '||', join(array2, '||')), '||')` | ✅ kept |

There is no built-in `merge()` or `combine()` function – `union()` and the join/concat/split pattern below cover both cases.

## ✅ Solution
Instead of `union`, concatenate the arrays with a unique delimiter and then split the result back into an array. This preserves every element, including duplicates.

## 🔧 How It's Done

1. **Join each array** into a string using a unique delimiter (`||`).

2. **Concatenate** both strings, keeping the delimiter between them.

3. **Split** the combined string back into an array using the same delimiter, all elements, duplicates included, are restored.

Here's the expression:

```
split(
  concat(
    join(outputs('Compose_-_2_arrays')?['array1'], '||'),
    '||',
    join(outputs('Compose_-_2_arrays')?['array2'], '||')
  ),
  '||'
)
```

## 🎉 Result
You merge both arrays while keeping all duplicate values, something the standard `union` function does not allow.

## 🌟 Key Advantages
🔸 Inclusivity: every element is kept, duplicates included.

🔸 Flexibility: adjust the formula for simple lists or more complex data structures.

🔸 Power: manipulate data in ways the standard functions don't directly allow.

---

## 🛠️ FAQ
**1. How do I combine two arrays in Power Automate?**

Use `union(array1, array2)` if duplicates may be removed, or the join/concat/split pattern above if every element must be kept.

**2. How do I merge two arrays without removing duplicates?**

`union()` always deduplicates. To keep duplicates use: `split(concat(join(array1, '||'), '||', join(array2, '||')), '||')`.

**3. What happens if my array contains values that include the delimiter '||'?**

Choose a unique delimiter that doesn't appear in your data, or use a more complex delimiter like '###DELIMITER###'.

**4. Can this method handle arrays with different data types?**

Yes, but ensure all values can be converted to strings for the join operation, then convert back to appropriate types if needed.

**5. Is there a performance impact when merging large arrays?**

For very large arrays, consider using the union() function for simple merging without duplicates, or batch processing for better performance.

---
