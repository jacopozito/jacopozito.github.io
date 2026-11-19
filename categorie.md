---
layout: page
title: Categories
permalink: /categories/
---

{% for category in site.categories %}
  {% assign category_name = category[0] %}
  {% assign posts = category[1] %}

  <h2 id="{{ category_name | slugify }}">{{ category_name | capitalize }} ({{ posts.size }})</h2>

  <ul>
    {% for post in posts %}
      <li>
        <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
        <small>({{ post.date | date: "%d/%m/%Y" }})</small>
      </li>
    {% endfor %}
  </ul>
{% endfor %}
