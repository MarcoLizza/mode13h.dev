---
title: 'tofu-engine #6'
author: marco.lizza
layout: post
permalink: "/tofu-engine-6"
comments: true
categories:
  - tofu-engine
  - devlog
tags:
  - graphics
  - optimization
  - sprite-batching
---
Holy Guacamole! It's been *seven* months since the last post. However, despite the long silence, **#tofuengine** developments haven't been suspended... very far from that!

In that first half (and a bit) of the year, there have been at the very least *two* significant developments, that is **audio-support** and **upgrading to Lua v5.4**. I took notes about the development and I plan to write more detailed posts. Sooner or later.

In the meantime, I'm getting back to the graphics sub-system of the engine to implement something that has been wandering in my mind since the very beginning. I put as much effort as I could to optimize the software-renderer and make *blitting* fast. Unless I switch and use OpenGL and leverage the GPU I don't think I can pour some FPS out of the low-level drawing functions.

But.... what if.

Probably there's room for some more optimization.

Let's take a step back.

At the moment the object's drawing operations are performed by calling the (overridden by arity) `blit()` method of the `Bank` class. If one wants to draw, let's say 10000 cute little bunnies on the screen, the method would be called 10000 times per frame. This means 10000 switches from the VM to the native C implementation. This isn't a piece of cake as the `blit()` method features up to 9 arguments.

Although the Lua VM does a pretty decent job in being fast when calling FFI function, this is nonetheless a waste of precious time.

Since the bunnies aren't changing their drawing parameters necessarily on each frame, we can track their drawing parameters by populating/updating a list of POD elements (stored in the FFI side were the `blit()` method implementation resides) on each fixed-update step... and then call a single `blit()` in the render method to (internally) scan the list and draw the objects on-screen. When we can clear the list and rebuild it with the new objects' parameters, or we can update only the changed ones... that's irrelevant. What it counts is that during the frame rendering the list is ready and we can draw the object with a single Lua call.

That's not a novel idea and it's commonly called [sprite-batching](https://randygaul.net/2014/01/01/simple-sprite-batching/) and it's used to optimized GPU-driven engines, mostly to minimize the number of texture swaps. In our case, there aren't textures to be swapped but we could benefit from it nevertheless since it should drastically reduce the overhead of the repeated draw calls.

That's enough for the talk. It's time for the design and implementation of that. More on this in the next post.
