---
layout: single
title: "Power Platform Newsletter - Weekly Tips & Tutorials | PowerPlatformTip"
permalink: /newsletter/
description: "Subscribe to the weekly PowerPlatformTip Newsletter by MVP Marcel Lehmann. Exclusive tips, tutorials and updates on Power Apps, Power Automate & Copilot Studio directly to your inbox."
excerpt: "Stay up to date with weekly Power Platform tips, best practices and news directly from Microsoft MVP Marcel Lehmann."
keywords: "PowerPlatformTips Newsletter, Power Platform, Marcel Lehmann, MVP, Power Apps Tutorial, Power Automate Tips, Copilot Studio"
author_profile: false
sitemap:
  priority: 0.8
  changefreq: weekly
header:
  overlay_color: "#38c9c3"
  overlay_filter: "0.2"
  overlay_image: "/assets/images/hero-bg.jpg"
  cta_label: "All Blog Posts"
  cta_url: "/posts/"
  cta_class: "btn--primary"
---

# 📧 Power Platform Tips Newsletter

Don’t miss any #PowerPlatformTips! Subscribe now to receive weekly:

- Exclusive practical tips from MVP Marcel Lehmann  
- Step-by-step tutorials and best practices  
- News on Power Apps, Power Automate, Dataverse & Copilot Studio  

<div class="newsletter-form" style="max-width: 560px; margin: 1.5rem auto;">
  <iframe
    src="https://f2405196.sibforms.com/serve/MUIFAKUg_sdL7qa_ISihRTrt0MaL-uPb9gnJAaPHjGGGqJIWcstC_jjbvid3Npe4v9adeNBuIGWwz_vPw2msuL_LzI5boEFv0KA-iV9PLJvvDtKfA1JWY-G2Qy6Z9tJP4hx7jRM7K4Q4qaSBmJFDIDhIOSrLqCa5Zke3aLODCk7aVKVBlGMKaBoB5nXvaSMD5n2rNrYHj677g9zI"
    title="Newsletter signup – PowerPlatformTip"
    style="width: 100%; height: 620px; border: 0; display: block; background: #fff; border-radius: 8px;"
    referrerpolicy="no-referrer-when-downgrade"></iframe>
</div>

---

*Thank you for your interest! Feel free to share this link so more developers can benefit from our weekly Power Platform Tips.*  
---

## 📰 Latest PowerPlatformTip Posts

<div style="display: flex; flex-wrap: wrap; gap: 2rem; align-items: flex-start; max-width: 1200px; margin: 0 auto;">
  <div style="flex: 2 1 600px; min-width: 0;">
    <ul class="post-list">
      {% for post in site.posts limit:3 %}
        <li>
          <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
          <span class="post-meta">{{ post.date | date: '%B %d, %Y' }}</span>
          <p>{{ post.excerpt | strip_html | truncate: 160 }}</p>
        </li>
      {% endfor %}
    </ul>
  </div>
</div>
