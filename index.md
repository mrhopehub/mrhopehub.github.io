---
layout: index
title: general code is good code
---
# {{ page.title }}
[Lofter](http://mrhopehub.lofter.com/)
## 文章列表

<ul>
    {% for post in site.posts %}
      <li>{{ post.date | date_to_string }} <a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>
