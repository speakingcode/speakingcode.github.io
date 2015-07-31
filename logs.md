---
layout: page
title: logs
permalink: /logs/
---

  <ul class="post-list">
    {% for post in site.posts %}
      <li>
        
        <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }} [ {{ post.date | date: "%b %-d, %Y" }} ] </a>
      </li>
    {% endfor %}
  </ul>
