---
layout: page
title: Slides
description: Slides from my talks.
permalink: /slides/
---

Slides from my talks. Use the arrow keys or scroll to move through a deck.

<ul class="blog-posts">
  {% for deck in site.data.slides %}
  <li>
    <span><i>{{ deck.when }}</i></span>
    <a href="{{ site.baseurl }}/slides/{{ deck.slug }}">{{ deck.title }}</a>
  </li>
  {% endfor %}
</ul>
