---
layout: page
title: "分类索引"
permalink: /categories/
---

<ul>
  {% for category in site.categories %}
    <li>
      <a href="/categories/{{ category[0] }}/">{{ category[0] }}</a> ({{ category[1].size }})
    </li>
  {% endfor %}
</ul>
