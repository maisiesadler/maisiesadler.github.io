---
layout: default
title: 'Maisie Sadler'
author: 'Maisie Sadler'
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
<!-- 
## Posts by tag

{% for tag in site.tags %}
  <h3>{{ tag[0] }}</h3>
  <ul>
    {% for post in tag[1] %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %} -->

## Notes

<ul>
  {% for n in site.notes %}
    <li>
      <a href="{{ n.url }}">{{ n.title }}</a> _{{n.about}}_
    </li>
  {% endfor %}
</ul>
