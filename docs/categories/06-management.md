---
layout: archive
permalink: /docs/management/
title: "Management"
---

{% assign posts = site.categories.Management %}
{% if posts %}
  {% for post in posts %}
    {% include archive-single.html type=entries_layout %}
  {% endfor %}
{% else %}
  {% include no-content.html %}
{% endif %}

