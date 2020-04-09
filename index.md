---
id: 137
title: List of All Articles
date: 2015-11-27T14:26:59-05:00
author: Sid
layout: test
guid: http://ducktypelabs.com/?page_id=137
cta_content_placement:
  - below
classic-editor-remember:
  - classic-editor
---
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
