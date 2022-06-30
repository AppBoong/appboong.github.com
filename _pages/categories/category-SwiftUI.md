---
layout: archive
permalink: /categories/SwiftUI
title: "Posts about SwiftUI"
author_profile: true
sidebar_main: true
search : false
---

{% assign posts = site.categories.SwiftUI | sort:"date" | reverse %}

{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}