---
layout: page
title: "标签索引"
permalink: /tags/
---

<ul>
  {% for tag in site.tags %}
    <li>
      <a href="/tags/{{ tag[0] }}/">{{ tag[0] }}</a> ({{ tag[1].size }})
    </li>
  {% endfor %}
</ul>
