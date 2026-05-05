---
layout: archive
title: "Blog – DevOps Notes"
permalink: /blog/
author_profile: true
---

{% include base_path %}

<h2>📓 Wszystkie notatki</h2>

<ul>
  {% for post in site.posts %}
    <li>
      <strong><a href="{{ post.url }}">{{ post.title }}</a></strong>
      <small><em>{{ post.date | date: "%Y-%m-%d" }}</em></small>
      <br>{{ post.excerpt | strip_html | truncatewords: 30 }}
    </li>
  {% endfor %}
</ul>