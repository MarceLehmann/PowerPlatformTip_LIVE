---
title: "Power Apps ComboBox DefaultSelectedItems: Set One or Multiple Default Values"
seo_title: "Power Apps ComboBox DefaultSelectedItems – Set Default Values"
date: 2021-04-26
permalink: "/article/powerplatform/2021/04/26/power-apps-combo-box-defaultselecteditems/"
updated: 2026-08-22
last_modified_at: 2026-08-22
categories:
  - Article
  - PowerPlatform
excerpt: "Set DefaultSelectedItems in a Power Apps ComboBox with copy-paste formulas: pre-select one or multiple items for text, Choice, Lookup and Person columns – plus fixes for the most common errors."
description: "How to set DefaultSelectedItems in a Power Apps ComboBox: working formulas to pre-select one or multiple default values for text, SharePoint Choice, Lookup and Person columns."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
  teaser: /assets/images/posts/external/ext_dfa092c519e3f5f951c7c56374e36f4a.jpg
toc: true
toc_sticky: true
tags:
  - Shane Young
  - PowerApps
  - ComboBox
  - DefaultSelectedItems
  - Controls
  - PowerPlatform
  - Tables
  - Lookup
  - YouTube
faq:
  - question: "Why does DefaultSelectedItems not work in my Power Apps ComboBox?"
    answer: "Almost always a type mismatch: DefaultSelectedItems expects a table of records with the same structure as the ComboBox Items property. Passing a plain text value like \"Option A\" fails – wrap it as a record, for example [{Value: \"Option A\"}]."
  - question: "How do I set multiple default values in a Power Apps ComboBox?"
    answer: "Return a table with several records, for example Filter(Choices(YourList.YourColumn), Value in [\"Option A\", \"Option B\"]) or a manual table like Table({Value: \"Option A\"}, {Value: \"Option B\"})."
  - question: "How do I set a default value for a SharePoint Choice column in a ComboBox?"
    answer: "Use [LookUp(Choices(YourList.YourColumn), Value = ThisItem.YourColumn.Value)] for a single default, or ThisItem.YourColumn directly for multi-select Choice columns when editing an existing item."
  - question: "How do I pre-select the current user in a Person column ComboBox?"
    answer: "Provide a record matching the person field schema, for example: [{Claims: \"i:0#.f|membership|\" & Lower(User().Email), DisplayName: User().FullName, Email: User().Email}]."
---

> **Quick answer:** `DefaultSelectedItems` expects a **table of records with the same structure as the ComboBox `Items` property**. Pre-select one item with `[LookUp(Choices(YourList.YourColumn), Value = "Option A")]`, multiple items with `Filter(Choices(YourList.YourColumn), Value in ["Option A", "Option B"])`, and for a simple text ComboBox `[{Value: "Option A"}]` is enough. A plain string like `"Option A"` will **not** work.

A great summary of how to deal with DefaultSelectedItems in a ComboBox and how to set them using a table/lookup or a manually-created record.

{% include video id="AXAbmy9zYTU" provider="youtube" %}

## Mastering ComboBox DefaultSelectedItems by Shane Young

This comprehensive tutorial by **Shane Young** tackles one of the most common challenges PowerApps developers face: properly configuring **DefaultSelectedItems** in ComboBox controls.

### The DefaultSelectedItems Challenge

ComboBox controls in PowerApps offer powerful multi-select capabilities, but setting default selections can be tricky. Common issues include:
- **Type mismatches** between data sources and default values
- **Collection formatting** problems
- **Lookup complexities** when working with related data
- **Performance issues** with large datasets

## 🎯 Shane's Comprehensive Approach

### Three Key Methods Covered

**1. Table-Based Defaults**
- Using existing data tables as the source
- Filtering records for default selection
- Maintaining data consistency

**2. Lookup-Based Defaults**
- Referencing related records
- Cross-table relationships
- Dynamic default selection

**3. Manually-Created Records**
- Building custom default collections
- Static default configurations
- Flexible data structures

## 💡 Understanding DefaultSelectedItems Property

### Data Type Requirements
The DefaultSelectedItems property expects:
```powerapps
// Correct format - collection of records
[
    {ID: 1, Title: "Option 1"},
    {ID: 2, Title: "Option 2"}
]

// NOT a simple text list
["Option 1", "Option 2"]  // ❌ Wrong format
```

### Common Patterns

**Pattern 1: Filter from Data Source**
```powerapps
Filter(
    YourDataSource,
    SomeCondition = true
)
```

**Pattern 2: Lookup Related Records**
```powerapps
LookUp(
    RelatedTable,
    ID = CurrentRecord.RelatedID
)
```

**Pattern 3: Manual Record Creation**
```powerapps
Table(
    {ID: 1, Value: "Default Option 1"},
    {ID: 2, Value: "Default Option 2"}
)
```

## 🔧 Implementation Strategies

### Strategy 1: Dynamic Defaults Based on User Context

```powerapps
// Set defaults based on current user's department
Filter(
    DepartmentOptions,
    Department = Office365Users.MyProfile().Department
)
```

### Strategy 2: Cascading ComboBox Defaults

```powerapps
// Child ComboBox defaults based on parent selection
Filter(
    ChildOptions,
    ParentID = ParentComboBox.Selected.ID
)
```

### Strategy 3: Multi-Table Lookup Defaults

```powerapps
// Combine data from multiple sources
ForAll(
    UserPreferences,
    LookUp(AvailableOptions, ID = PreferenceID)
)
```

## 🚀 Advanced Techniques

### Performance Optimization

**Technique 1: Pre-loaded Collections**
```powerapps
// On app start, create optimized collections
ClearCollect(
    DefaultSelections,
    AddColumns(
        UserDefaults,
        "FullRecord",
        LookUp(MasterData, ID = DefaultID)
    )
)
```

**Technique 2: Conditional Loading**
```powerapps
// Load defaults only when needed
If(
    !IsBlank(CurrentUser),
    Filter(Defaults, UserID = CurrentUser.ID),
    Blank()
)
```

### Error Handling

**Robust Default Setting**
```powerapps
// Handle missing or invalid defaults gracefully
IfError(
    Filter(DataSource, IsDefaultSelection),
    Table({ID: 0, Title: "No Default Available"})
)
```

## 📊 Real-World Use Cases

### Business Applications

**Employee Department Selection**
- Default to user's current department
- Allow selection of additional departments
- Maintain organizational hierarchy

**Project Assignment**
- Default to user's active projects
- Enable multi-project selection
- Filter by project status

**Product Categories**
- Default based on user preferences
- Support category hierarchies
- Enable bulk selection

### Technical Considerations

**Data Consistency**
- Ensure default records exist in the data source
- Handle deleted or archived records
- Validate record structure matches ComboBox items

**User Experience**
- Provide clear visual feedback for defaults
- Allow easy modification of selections
- Maintain selection state across sessions

## 🎯 Best Practices from Shane's Tutorial

### Development Guidelines

1. **Always validate data types** between defaults and ComboBox items
2. **Use collections** for complex default logic
3. **Test with realistic data volumes** to ensure performance
4. **Implement error handling** for missing or invalid defaults
5. **Consider user context** when setting defaults

### Performance Tips

- **Pre-load static defaults** in collections
- **Use delegation-friendly** filtering where possible
- **Avoid complex lookups** in DefaultSelectedItems when possible
- **Cache frequently used** default configurations

## 🔄 Troubleshooting Common Issues

### Issue 1: Type Mismatch Errors
**Solution**: Ensure DefaultSelectedItems returns the same record structure as ComboBox Items

### Issue 2: Performance Problems
**Solution**: Use collections and pre-filtered data instead of complex real-time queries

### Issue 3: Empty Defaults
**Solution**: Implement fallback logic with proper error handling

### Issue 4: Delegation Warnings
**Solution**: Move complex logic to collections created on app start

## 💼 Enterprise Implementation

### Scaling Considerations
- **Centralized default logic** in component libraries
- **Configuration-driven** default settings
- **User preference** management systems
- **Audit trails** for default selections

### Security Aspects
- **Row-level security** for default records
- **User permission** validation
- **Data governance** compliance
- **Privacy considerations** for user defaults

## 🎖️ About Shane Young

Shane Young continues to demonstrate expertise in:
- **Practical PowerApps development** techniques
- **Real-world problem solving**
- **Performance optimization** strategies
- **User experience** improvements

This DefaultSelectedItems tutorial showcases Shane's ability to break down complex concepts into actionable, implementable solutions.

## 🎯 Key Takeaways

- **Understand data type requirements** for DefaultSelectedItems
- **Use appropriate patterns** based on your data structure
- **Implement proper error handling** for robust applications
- **Consider performance implications** of default selection logic
- **Test thoroughly** with realistic data scenarios
- **Follow delegation best practices** for scalable solutions

This comprehensive guide ensures developers can confidently implement DefaultSelectedItems in any ComboBox scenario, from simple static defaults to complex dynamic multi-table lookups.

## 🛠️ FAQ

**1. Why does DefaultSelectedItems not work in my ComboBox?**

Almost always a type mismatch: `DefaultSelectedItems` expects a **table of records** with the same structure as the ComboBox `Items` property. Passing a plain text value like `"Option A"` fails – wrap it as a record: `[{Value: "Option A"}]`.

**2. How do I set multiple default values in a Power Apps ComboBox?**

Return a table with several records:

```powerapps
Filter(Choices(YourList.YourColumn), Value in ["Option A", "Option B"])
// or a manual table
Table({Value: "Option A"}, {Value: "Option B"})
```

**3. How do I set a default value for a SharePoint Choice column?**

```powerapps
// Single-select Choice column
[LookUp(Choices(YourList.YourColumn), Value = ThisItem.YourColumn.Value)]
// Multi-select Choice column (edit form)
ThisItem.YourColumn
```

**4. How do I pre-select the current user in a Person column ComboBox?**

```powerapps
[{
    Claims: "i:0#.f|membership|" & Lower(User().Email),
    DisplayName: User().FullName,
    Email: User().Email
}]
```

---

*You can see this video here on my blog because I have rated this video with 5 stars in my YouTube video library. This video was automatically posted using PowerAutomate.*
