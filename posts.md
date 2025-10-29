---
layout: default
title: Posts
---

{% include nav.html %}
<link rel="stylesheet" href="/assets/style.css">

# Publicaciones

<div style="display: flex; flex-wrap: wrap; gap: 20px; padding: 20px 0;">
  {% for post in site.posts %}
  <div style="
      border: 1px solid #ddd;
      border-radius: 8px;
      padding: 15px;
      width: calc(33% - 20px);
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
      background-color: #fff;
    ">
    <h3 style="margin-top:0;">
      <a href="{{ site.baseurl }}{{ post.url }}" style="color: #0d6efd; text-decoration: none;">{{ post.title }}</a>
    </h3>
    <p style="color:#777; font-size: 14px;">{{ post.date | date: "%Y-%m-%d" }}</p>
    <p>{{ post.excerpt | strip_html | truncate: 120 }}</p>
  </div>
  {% endfor %}
</div>


