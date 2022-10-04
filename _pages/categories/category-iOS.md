---
layout: archive
permalink: /categories/iOS
title: "Posts about iOS"
author_profile: true
sidebar_main: true
search : false
---

{% assign posts = site.categories.iOS | sort:"date" | reverse %}

{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}