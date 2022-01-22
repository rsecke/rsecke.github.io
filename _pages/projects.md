---
title: Projects
layout: collection
permalink: /projects/
collection: portfolio
entries_layout: grid
---

{% include base_path %}

<div class="grid__wrapper">
  {% for post in site.pages.projects %}
    {% include archive-single.html type="grid" %}
  {% endfor %}
</div>