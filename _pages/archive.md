---
title: Posts Archive
author: marco.lizza
layout: page
permalink: /archive/
comments: false
hide_meta: false
published: true
---
{% for post in site.posts %}
  {% capture current_year %}{{ post.date | date: "%Y" }}{% endcapture %}
  {% if current_year != previous_year %}
## {{ current_year }}
  {% endif %}
* [{{ post.title }}]({{ post.url | prepend: site.baseurl | prepend: site.url  }})
  {% capture previous_year %}{{ current_year }}{% endcapture %}
{% endfor %}