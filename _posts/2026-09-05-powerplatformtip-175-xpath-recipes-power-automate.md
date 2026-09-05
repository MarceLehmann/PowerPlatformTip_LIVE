---
title: "#PowerPlatformTip 175: '25 Everyday xpath() Recipes That Kill Apply to each in Power Automate'"
date: 2026-09-05
categories:
  - Article
  - PowerPlatformTip
tags:
  - PowerAutomate
  - Expressions
  - XPath
  - XML
  - JSON
  - Performance
  - PowerPlatformTip
excerpt: "Stop nesting Apply to each, Filter Array and Increment variable for simple list work. One xpath() expression can count, sum, filter, pluck a single value or find the max over a JSON array in a single action, using the .NET XPath engine built into Power Automate."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
---

> **TL;DR:** Convert any JSON array to XML with `xml(json(...))`, then run one `xpath()` expression to count, sum, average, filter, pluck or rank data. It replaces whole `Apply to each` + `Filter Array` + `Increment variable` blocks with a single, delegation-free action.

Most flows lean on `Apply to each` for jobs that never needed a loop: counting matches, adding up a column, grabbing one field from one record, or filtering a few rows. Every loop iteration is an action run, and nested loops explode your run history and runtime.

Power Automate ships the `xpath()` function, which runs on the **.NET XPath 1.0 library** in both Consumption and Standard workflows. Feed it XML and an XPath expression and it returns the matching nodes or a computed value in **one** action. Below are 25 everyday recipes you can paste straight into the expression editor.

## 💡 Challenge
Simple data questions turn into oversized flows: an `Apply to each` with an `Increment variable` to count, another loop to total an amount, a `Filter Array` plus `First()` to read one field. Each pattern adds actions, action-run cost and, on large collections, delegation and timeout headaches, for logic that is really a one-liner.

## ✅ Solution
`xpath()` navigates an XML document and can **select nodes** *and* **compute values** (`count()`, `sum()`, `string()`, `contains()`, …) in a single expression. Since your data is usually JSON, first turn it into XML with the built-in `xml()` and `json()` functions, then query it. No loops, no extra variables, no premium connector.

## 🔧 How it's done

### Step 0 – Turn your JSON array into XML (do this once)

`xml()` needs a single **root** object, so wrap your array under a root and an item name. Put this in a **Compose** action (call it `To_XML`):

```
xml(json(concat('{"root":{"item":', string(outputs('Get_items')?['body/value']), '}}')))
```

That produces XML like:

```xml
<root>
  <item><ID>1</ID><Customer>Gala Ltd</Customer><Country>CH</Country><Status>Open</Status><Amount>1200</Amount><Email>a@gala.ch</Email></item>
  <item><ID>2</ID><Customer>Rio SA</Customer><Country>BR</Country><Status>Paid</Status><Amount>430</Amount><Email>b@rio.br</Email></item>
  ...
</root>
```

Every recipe below is used as `xpath(outputs('To_XML'), '<expression>')`. Only the `<expression>` changes.

### The 25 recipes

| # | Everyday task | XPath expression | What it saves |
|---|---------------|------------------|---------------|
| 1 | Count all records | `count(/root/item)` | `Apply to each` + `Increment variable` |
| 2 | Count matching records | `count(/root/item[Status="Open"])` | Loop + condition + counter |
| 3 | Sum a column | `sum(/root/item/Amount)` | Loop + `add()` on a variable |
| 4 | Conditional sum (e.g. per country) | `sum(/root/item[Country="CH"]/Amount)` | `Filter Array` + loop + sum |
| 5 | Average of a column | `sum(/root/item/Amount) div count(/root/item)` | Two loops (sum + count) |
| 6 | All values of one field as an array | `/root/item/Email/text()` | `Select` action |
| 7 | First record's field | `string(/root/item[1]/Customer)` | `first()` + property access |
| 8 | Last record's field | `string(/root/item[last()]/Customer)` | `last()` + `length()` math |
| 9 | Nth record's field | `string(/root/item[3]/Customer)` | Index gymnastics |
| 10 | One field from a keyed record | `string(/root/item[ID="42"]/Email)` | `Filter Array` + `First()?['Email']` |
| 11 | Filter rows by exact value | `/root/item[Status="Open"]` | `Filter Array` |
| 12 | Numeric threshold filter | `/root/item[number(Amount)>1000]` | `Filter Array` (advanced mode) |
| 13 | Two conditions (AND) | `/root/item[Status="Open" and number(Amount)>500]` | Nested `Filter Array` |
| 14 | Two conditions (OR) | `/root/item[Status="Open" or Status="Pending"]` | Two `Filter Array` + `union()` |
| 15 | "Contains" text filter | `/root/item[contains(Customer,"Ltd")]` | `Filter Array` with `contains()` |
| 16 | "Starts with" filter | `/root/item[starts-with(Email,"a")]` | `Filter Array` with `startsWith()` |
| 17 | Existence check (boolean) | `boolean(/root/item[Email="a@gala.ch"])` | Loop + found-flag variable |
| 18 | Max value's record (no `max()` in XPath 1.0) | `/root/item[not(../item/number(Amount) > number(Amount))]/Customer` | Sort + `first()` |
| 19 | Min value's record | `/root/item[not(../item/number(Amount) < number(Amount))]/Customer` | Sort + `first()` |
| 20 | Read an attribute | `string(/root/item[1]/@id)` | Manual JSON parse |
| 21 | Split an email domain | `substring-after(string(/root/item[1]/Email),"@")` | `split()` + index |
| 22 | Join first + last into full name | `concat(string(/root/item[1]/First)," ",string(/root/item[1]/Last))` | `concat()` on two lookups |
| 23 | Trim / normalize whitespace | `normalize-space(string(/root/item[1]/Customer))` | `trim()` chains |
| 24 | Skip header / first row | `/root/item[position()>1]` | `skip()` on an array |
| 25 | Namespaced XML (SOAP/SharePoint) | `/*[local-name()="Envelope"]/*[local-name()="Body"]` | Fighting `xmlns` prefixes |

> Tip: when a numeric comparison misbehaves, wrap the field in `number(...)` (recipes 12, 13, 18, 19). XPath compares strings unless you force a number. For text values wrap in `string(...)` so you get a clean value instead of an XML node.

### Three worked examples

**A) Count open orders – 1 action instead of a loop**

```
xpath(outputs('To_XML'), 'count(/root/item[Status="Open"])')
```
Returns an integer, e.g. `7`. No `Apply to each`, no counter variable.

**B) Total revenue for Switzerland only**

```
xpath(outputs('To_XML'), 'sum(/root/item[Country="CH"]/Amount)')
```
Filters and sums in one pass, replacing a `Filter Array` + loop + running total.

**C) Find the customer with the highest amount (the `max()` trick)**

XPath 1.0 has no `max()`, so select the item that **nothing exceeds**:

```
xpath(outputs('To_XML'), '/root/item[not(../item/number(Amount) > number(Amount))]/Customer/text()')
```
Returns the top customer(s) without sorting the array first.

## 🎉 Result
A handful of everyday reporting and lookup patterns collapse from multi-action, sometimes nested, loops into a single `xpath()` expression. Flows get shorter, cheaper in action runs, easier to read, and they sidestep the delegation and timeout pain of looping over large collections, all with standard actions only.

## 🌟 Key Advantages

🔸 **No loops:** count, sum, filter and rank run in one action instead of `Apply to each`.

🔸 **Standard, non-premium:** `xpath()`, `xml()` and `json()` are built-in expression functions.

🔸 **Compute *and* select:** the same call can return a value (`count`, `sum`) or a node set (filtered rows).

🔸 **Fewer action runs:** less run-history noise, lower cost, faster execution on big arrays.

## 🛠️ FAQ

**Q1: Why do I have to wrap my array in `{"root":{"item": ... }}`?**

`xml()` converts a single JSON **object**, not a bare array. Wrapping under `root`/`item` gives every element a predictable node name (`/root/item`) to query.

**Q2: My comparison like `Amount>1000` returns the wrong rows. Why?**

XPath compares as text by default. Force numeric context with `number(Amount)>1000`. For string results, wrap the selection in `string(...)` so you get the value, not an XML node.

**Q3: The XML looks wrong when the array has exactly one item.**

With a single element `xml()` can produce a flatter shape. Guard for it (e.g. check `count(/root/item)`), or ensure your source always returns an array before converting.

**Q4: Is there a `max()`, `min()` or `distinct-values()` function?**

No, `xpath()` uses XPath **1.0**, which has no `max()`, `min()` or `distinct-values()`. Use the `not(... > ...)` / `not(... < ...)` trick (recipes 18–19) for max/min, and average via `sum() div count()` (recipe 5).

**Q5: Do I need to escape the double quotes in my expression?**

In the expression editor, no, quotes work as shown. In **code view** (raw JSON definition) escape inner double quotes with a backslash, e.g. `//name[@expired=\"true\"]`.

## 🔗 Related Tips
- [#PowerPlatformTip 3: Advanced Filtering Array](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-3-advanced-filtering-array/), when a real `Filter Array` fits better than XPath.
- [#PowerPlatformTip 158: Return Flow Data to Power Apps with ParseJSON](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-158-parsejson-return-flow-data-power-apps/), the JSON-side companion for reshaping data.
- [Extract HTML Table from Email in Power Automate: Advanced XML Processing](https://www.powerplatformtip.com/article/powerplatform/2022/11/10/extract-html-table-email-power-automate/), XPath applied to real HTML email tables.
