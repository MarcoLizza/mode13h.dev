---
title: 'Custom Memory Allocators'
author: marco.lizza
layout: post
permalink: "/custom-memory-allocators"
comments: true
categories:
  - tofu-engine
  - rants
tags:
  - engine
  - redesign
  - memory
---
Memory allocators.

*Custom* memory allocators.

This is something I'm perpetually gravitating around, as I always loved them.

This might be due to the fact that in the '80s and early '90s we had **no** memory allocators at all, so we used to implement our custom one for each and every program. Having full control over how the memory was used was a breeze. It was ecstatic. However, this is not something one would do today, unless one is working on very specific embedded platforms very close to the bare-metal.

*( the temptation to just allocate a *huge* and *fixed* amount of memory during the program startup and just cope with it is strong, however... )*

So, why *#tofuengine* isn't using a custom memory allocator?

First and foremost, GCC's default allocator isn't that bad. Despite not being specifically designed for real-time situations, it's performant *enough*. Since game resources aren't continually allocated/freed over the lifespan of the game (or, at least, that scene/stage/level), using a more performant allocator it won't bring noticeable boost. In case more sophisticated management is required, it would be the resource manager task to cope with it, and not to just to put our hopes on a different/better memory allocator.

Moreover, while the main chunk of the game engine code can easily adapted to use a custom/different allocator, a subset of the used libraries (e.g. `libxmp`) heavily depends on the standard memory allocator (i.e.  `malloc()`). We could rewrite/extend/adapt those libraries (and from a design perspective it's something that *any* external library should support, along with custom I/O support), but it's something that it's on a lower priority level at the moment.

So, in the end, for **#tofuengine** I concluded that there's no point in trying and use a memory allocator different than the standard GCC provided one.

For the moment being. :)
