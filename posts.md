---
layout: default
title: Posts
---

{% include nav.html %}

# Publicaciones

<ul>
{% for post in site.posts %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a> — {{ post.date | date: "%Y-%m-%d" }}
  </li>
{% endfor %}
</ul>
