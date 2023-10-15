---
layout: archive
permalink: /docs/new-trend/
title: "New Trends"
---

{% assign posts = site.categories['New Trend'] %}
{% if posts %}
  {% for post in posts %}
    {% include archive-single.html type=entries_layout %}
  {% endfor %}
{% else %}
  {% include no-content.html %}
{% endif %}
