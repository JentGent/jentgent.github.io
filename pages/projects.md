---
layout: page
title: Projects
permalink: /projects/
feature-img: "img/color.png"
order: 4
---

<div class="work">
  <!-- <h2>Videos</h2> -->
  {% for project in site.portfolio %}
    {% if project.type == "video" %}
      <div class="project">
        <a class="project-thumb" href="{{ project.url }}">
          <img src="{{ project.thumbnail-path }}" alt="{{ project.title }}"/>
          <strong>{{ project.title }}</strong>
        </a>
        <div class="project-description">
          <p>{{ project.short-description }}</p>
        </div>
      </div>
    {% endif %}
  {% endfor %}
</div>
