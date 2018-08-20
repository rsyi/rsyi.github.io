---
layout: page
title: Archive
permalink: /archive/
---

<div class="posts">
  {% for post in site.posts %}
  {{ post.date | date: "%B %d, %Y" }}: <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a><br>
  {% endfor %}
</div>
