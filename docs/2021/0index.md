---
title: 2021
nav_order: 2021
description: "Blogs"
permalink: docs/2021
has_children: true
search_exclude: true
---

# Blogs for 2021

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
