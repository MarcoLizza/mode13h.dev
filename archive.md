---
layout: default
title: Archive
permalink: /archive/
---
<div class="page-content wc-container">
  <h1>Archive</h1>
  {% for post in site.posts %}
    {% capture current_year %}{{ post.date | date: "%Y" }}{% endcapture %}
    {% if current_year != previous_year %}
    {% if forloop.first == false %}</ul>{% endif %}
  <h5>{{ current_year }}</h5>
  <ul class="posts">
    {% endif %}
    <li><a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a></li>
    {% if forloop.last %}</ul>{% endif %}
    {% capture previous_year %}{{ current_year }}{% endcapture %}
  {% endfor %}
</div>