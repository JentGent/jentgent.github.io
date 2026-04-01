---
layout: page
title: Demos
permalink: /demos/
feature-img: "img/color.png"
order: 2
---

<div class="work">
  {% assign demos = site.demos | sort: "order" %}
  {% for project in demos %}
  <div class="project">
    <a class="project-thumb" style="width: 300px;" href="{{ project.url }}">
      <img src="{{ project.thumbnail-path }}" alt="{{ project.title }}"/>
      <strong>{{ project.title }}</strong>
    </a>
    <div class="project-description">
      <p>{{ project.short-description }}</p>
    </div>
  </div>
  {% endfor %}
</div>
