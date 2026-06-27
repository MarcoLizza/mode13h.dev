---
title: Tofu Engine
layout: page
permalink: /tofu-engine/
banner: /assets/images/tofuengine-logo_with_payoff-512x160.png
---
In my *#gamedev* career I worked on a lot of tools and libraries. However, despite having *completed* some games (which is something unusual for programmers, since we tend to switch interest frequently just because *there is something else very interesting to test out*) I've never written a full-fledged game-engine in the strict sense. It has always been something bound and specific to the game itself, and not generic enough to be re-used for other projects.

So, I resumed the old desire of mine to write my own engine. It's called **Tofu Engine** because... why not? :)

For the curious ones, here is the list of posts documenting its development.

{% for post in site.posts %}
  {% if post.categories contains "tofu-engine" %}
* {{ post.date | date: "%d %B %Y" }} - [{{ post.title }}]({{ post.url | relative_url }})
  {% endif %}
{% endfor %}
