---
layout: default
title: 'Maisie Sadler'
---

## Projects

- [Lively 🌳](https://maisiesadler.github.io/lively/)

## Posts

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
