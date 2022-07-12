---
layout: archive
permalink: /categories/Swift
title: "Posts about Swift"
author_profile: true
sidebar_main: true
search : false
---

{% assign posts = site.categories.Swift | sort:"date" | reverse %}

{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}