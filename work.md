---
layout: page
title: Work
description: Things I've shipped, at ADEO and on my own.
permalink: /work
---

Things I've shipped recently.

<ul class="blog-posts">
  {% assign projects = site.work | sort: 'order' %}
  {% for project in projects %}
  <li>
    <span><i>{{ project.where }}</i></span>
    <a href="{{ site.baseurl }}{{ project.url }}">{{ project.title }}</a>
  </li>
  {% endfor %}
</ul>
