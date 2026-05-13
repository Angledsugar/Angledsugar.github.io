---
layout: page
title: projects
permalink: /projects/
description: Research and hardware projects.
nav: true
nav_order: 3
display_categories: [research, hardware]
---

<!-- pages/projects.md -->
<div class="publications">
{% if site.enable_project_categories and page.display_categories %}
  {% for category in page.display_categories %}
    <h2 class="category"><a id="{{ category }}" href=".#{{ category }}" style="color: inherit;">{{ category }}</a></h2>
    {% assign categorized_projects = site.projects | where: "category", category %}
    {% assign sorted_projects = categorized_projects | sort: "importance" %}
    <ol class="bibliography">
      {% for project in sorted_projects %}
        {% include project_entry.liquid %}
      {% endfor %}
    </ol>
  {% endfor %}
{% else %}
  {% assign sorted_projects = site.projects | sort: "importance" %}
  <ol class="bibliography">
    {% for project in sorted_projects %}
      {% include project_entry.liquid %}
    {% endfor %}
  </ol>
{% endif %}
</div>
