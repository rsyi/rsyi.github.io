---
layout: page
title: Archive
permalink: /archive/
---

<div class="posts">
  {% for post in site.posts %}
    <article class="post">
      <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
    </article>
  {% endfor %}
</div>
