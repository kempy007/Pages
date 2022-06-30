---
title: 2022
nav_order: 2022
description: "Blogs"
permalink: docs/2022
has_children: true
search_exclude: true
---

# Blogs for 2022

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

{% for tag in site.tags %}
  <h3>{{ tag[0] }}</h3>
  <ul>
    {% for post in tag[1] %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}
