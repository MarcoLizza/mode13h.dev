---
title: Tofu Engine
author: marco.lizza
layout: page
permalink: /tofu-engine/
comments: true
banner: /assets/images/tofuengine-logo_with_payoff-512x160.png
---
In my *#gamedev* career I worked on a lot of tools and libraries. However, despite having *completed* some games (which is something unusual for programmers, since we tend to switch interest frequently just because *there is something else very interesting to test out*) I've never written a full-fledged game-engine in the strict sense. It's been always something bound and specific to the game itself, and not generic enough to be re-used for other projects.

So, I resumed the old desire of mine to write my own engine. It's called **Tofu Engine** because... why not? :)

For the courious ones, here's you'll find the list of post documenting its development.

{% for post in site.posts %}
  {% if post.categories contains "tofu-engine" %}
* {{ post.date | date: "%d %B %Y" }} - [{{ post.title }}]({{ post.url | prepend: site.baseurl | prepend: site.url  }})
  {% endif %}
{% endfor %}
