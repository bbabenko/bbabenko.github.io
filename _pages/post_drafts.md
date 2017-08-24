---
layout:       page
title:        Drafts
permalink:    /drafts.html
meta_robots:  noindex
---

<h1>blog posts</h1>
<ul>
  {% for post in site.post_drafts %}
    <li><span>{{ post.date | date: "%Y-%m-%d" }}</span> - <a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
