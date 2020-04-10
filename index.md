---
id: 137
title: About
date: 2015-11-27T14:26:59-05:00
author: Sid
layout: default
guid: http://ducktypelabs.com/?page_id=137
---
I'm Sid Krishnan, software developer and author of all the articles on this website.

For more information, check out [my linkedin profile](https://www.linkedin.com/in/siddharth-krishnan-b2a79a8) or [get in touch!](mailto://sidk@ducktypelabs.com)

## List of Articles
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
