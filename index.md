---
id: 137
title: List of All Articles
date: 2015-11-27T14:26:59-05:00
author: Sid
layout: default
guid: http://ducktypelabs.com/?page_id=137
---
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
