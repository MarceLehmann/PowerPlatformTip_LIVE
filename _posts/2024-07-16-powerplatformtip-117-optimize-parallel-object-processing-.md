---
title: "#PowerPlatformTip 117: 'Optimize Parallel Object Processing'"
date: 2024-07-16
last_modified_at: 2026-07-20
categories:
  - Article
  - PowerPlatformTip
tags:
  - PowerAutomate
  - Concurrency
  - ParallelProcessing
  - ChildFlows
  - Performance
  - PowerPlatformTip
excerpt: "Speed up large-item processing in Power Automate by leveraging child flows, database triggers, and strategic 'Respond to a PowerApp or flow' placement."
header:
  overlay_color: "#2dd4bf"
  overlay_filter: "0.5"
toc: true
toc_sticky: true
faq:
  - question: "How do I set concurrency in Apply to Each?"
    answer: "Adjust the concurrency control in the \"Apply to each\" action settings up to a maximum of 50 parallel runs. Beyond that, use child flows or database triggers."
  - question: "What are child flows and why use them?"
    answer: "Child flows are separate flows invoked from a parent flow. They let you process items independently and in parallel, bypassing the parent flow's concurrency limits."
  - question: "How do I handle errors in child flows?"
    answer: "Use scopes and configure run-after settings inside the child flow. Log failures to a database or send notifications, and optionally retry or resubmit failed runs later."
---

> **TL;DR:** Break past the 50-item Apply to Each limit in Power Automate using child flows or database triggers for true parallel processing.

## 💡 Challenge
Processing many items in Power Automate's "Apply to each" can be slow. Even with concurrency on, you're capped at 50 parallel runs, so later items wait for a slot to free up.

## ✅ Solution
Trigger child flows or write items to a database so each item is processed independently, and place "Respond to a PowerApp or flow" before long operations so the main flow returns quickly.

## 🔧 How It's Done

**1. Trigger a child flow per item**

🔸 Enables parallel execution beyond the 50-item cap.

🔸 Create a child flow that handles a single item, then call it for each item from the main flow.

**2. Write to a database**

🔸 Keeps data integrity with controlled, structured processing.

🔸 Write each item to a database inside the loop, then use a second flow that triggers on new rows to process them individually.

**3. Place "Respond to a PowerApp or flow" early**

🔸 Lets the flow return quickly and frees the system for more requests.

🔸 Put the Respond action before time-intensive steps; resubmit failed runs later if needed.

## 🎉 Result
Large item sets process significantly faster, bypassing the default 50-item concurrency limit and improving overall flow responsiveness.

## 🌟 Key Advantages

🔸 **Speed:** far less time to process large item counts.

🔸 **Scale:** parallel processing beyond the 50-item limit.

🔸 **Control:** structured processing via child flows or databases.

---

## 🎥 Video Tutorial
{% include video id="p6iVVVzgD5A" provider="youtube" %}

---

## 🛠️ FAQ
**1. How do I set concurrency in Apply to Each?**
Adjust the concurrency control in the "Apply to each" action settings up to a maximum of 50 parallel runs. Beyond that, use child flows or database triggers.

**2. What are child flows and why use them?**
Child flows are separate flows invoked from a parent flow. They let you process items independently and in parallel, bypassing the parent flow's concurrency limits.

**3. How do I handle errors in child flows?**
Use scopes and configure run-after settings inside the child flow. Log failures to a database or send notifications, and optionally retry or resubmit failed runs later.

## 🔗 Related Tips
- [#PowerPlatformTip 75: Boost Efficiency with Concurrency Control](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-75-boost-efficiency-with-concurrency-control/), tune the Apply to each concurrency setting.
- [#PowerPlatformTip 131: ForAll & Patch Optimization in PowerApps](https://www.powerplatformtip.com/article/powerplatformtip/powerplatformtip-131-forall-patch-optimization-in-powerapps/), optimise bulk item operations.
