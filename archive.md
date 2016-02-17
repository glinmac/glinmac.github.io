---
layout: page
title: Archive
---

<ul>
  {% for post in site.posts %}
    <li>
      {{ post.date | date: "%Y-%m-%d"}} &#9656; <a href="{{ post.url }}">{{ post.title }}</a> <small>[{{ post.categories | join: ', ' }}]</small>
    </li>
  {% endfor %}
</ul>
