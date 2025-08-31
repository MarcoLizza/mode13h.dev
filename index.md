---
layout: default
---
<div class="command-line">
  <span class="prompt">{{ site.prompt }}</span>
  <span class="command">ls -la pages</span>
</div>
<ul class="post-list">
  {% assign pages = site.pages | where_exp: 'page', 'page.title' | sort: "title" %}
  {% for page in pages %}
    {% if page.navigation != false %}
      <li class="post-list-item">
        <span class="permissions">-rw-r--r--</span>
        <span class="author">{{ page.author | default: site.author.username }}</span>
        <span class="date">{{ page.date | default: "1970-01-01" | date: "%Y-%m-%d" }}</span>
        <a href="{{ page.url | relative_url }}">{{ page.title }}</a>
      </li>
    {% endif %}
  {% endfor %}
</ul>

<div class="command-line">
  <span class="prompt">{{ site.prompt }}</span>
  <span class="command">ls -la posts</span>
</div>
<ul class="post-list">
  {% assign posts = site.posts | where_exp: 'post', 'post.title' %}
  {% for post in posts limit: site.recent_posts_limit %}
    <li class="post-list-item">
        <span class="permissions">-rw-r--r--</span>
        <span class="author">{{ post.author | default: site.author.username }}</span>
        <span class="date">{{ post.date | default: "1970-01-01" | date: "%Y-%m-%d" }}</span>
        <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
