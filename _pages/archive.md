---
title: Posts Archive
author: marco.lizza
layout: page
permalink: /archive/
comments: false
hide_meta: false
published: true
---
<div class="page-content wc-container">
  {% for post in site.posts %}
    {% capture current_year %}{{ post.date | date: "%Y" }}{% endcapture %}
    {% if current_year != previous_year %}
  <h5>{{ current_year }}</h5>
      {% if forloop.first == false %}</ul>{% endif %}
  <ul class="posts">
    {% endif %}
    <li><a href="{{ post.url | prepend: site.baseurl | prepend: site.url  }}">{{ post.title }}</a></li>
    {% if forloop.last %}</ul>{% endif %}
    {% capture previous_year %}{{ current_year }}{% endcapture %}
  {% endfor %}
</div>