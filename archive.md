---
layout: default
title: Posts Archive
---
<div class="page-content wc-container">
  <h1>Posts Archive</h1>  
  {% for post in site.posts %}
    {% capture current_year %}{{ post.date | date: "%Y" }}{% endcapture %}
    {% if current_year != previous_year %}
  <h5>{{ current_year }}</h5>
  <ul class="posts">
    {% endif %}
    <li><a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a></li>
    {% if forloop.last %}</ul>{% endif %}
    {% capture previous_year %}{{ current_year }}{% endcapture %}
  {% endfor %}
</div>