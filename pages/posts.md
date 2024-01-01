---
layout: page
title: Posts
permalink: "/posts/"
feature-img: "img/color.png"
order: 3
---

<ul class="posts">
  {% for post in site.posts %}
    {% if post.hidden != true %}
      <li class="post-teaser">
        <header>
          <h3>
            <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">
              {% if post.feature-img %}
                <img src="{{ post.feature-img }}">
              {% endif %}
              {{ post.title }}
            </a>
          </h3>
          <p class="meta">
            {{ post.date | date: "%B %-d, %Y" }}
          </p>
        </header>
        <div class="excerpt">
          {{ post.excerpt | strip_html | strip_newlines | truncate: 120 }}
        </div>
        <a href="{{ post.url | prepend: site.baseurl }}">
          {{ site.theme_settings.str_continue_reading }}
        </a>
      </li>
    {% endif %}
  {% endfor %}
</ul>