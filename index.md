---
layout: default
title: Inicio
---

{% include nav.html %}
<link rel="stylesheet" href="/assets/style.css">

<div style="text-align:center; padding:60px 20px;">
  <h1 style="font-size:42px; font-weight:700;">📊 Blog BAABD</h1>
  <p style="font-size:20px; color:#555;">
    Proyectos y prácticas de Big Data, Spark y Analítica
  </p>
</div>

## 📌 Últimas publicaciones
<ul style="list-style:none; padding-left:0;">
{% for post in site.posts %}
  <li style="margin-bottom:20px;">
    <a href="{{ site.baseurl }}{{ post.url }}" style="font-size:18px; font-weight:600;">{{ post.title }}</a>
    <br>
    <span style="color:#777;">Publicado el {{ post.date | date: "%Y-%m-%d" }}</span>
  </li>
{% endfor %}
</ul>

---

<div style="text-align:center; margin-top:40px;">
  <p>Desarrollado por <strong>Deison Padilla</strong> — BAABD 🚀</p>
</div>
