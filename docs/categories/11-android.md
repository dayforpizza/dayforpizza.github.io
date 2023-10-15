---
layout: archive
permalink: /docs/android/
title: "Android"
---

{% assign posts = site.categories.Android %}
{% if posts %}
  {% for post in posts %}
    {% include archive-single.html type=entries_layout %}
  {% endfor %}
{% else %}
  {% include no-content.html %}
{% endif %}
