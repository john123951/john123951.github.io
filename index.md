---
layout: page
title: 不存在的完美
tagline: 知识才是一个程序员最虔诚的信仰
---
{% include JB/setup %}

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; 
    <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a>
    <p>{{post.description}}</p>
    </li>
  {% endfor %}
</ul>