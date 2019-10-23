---
title: 'tofu-engine #4'
author: marco.lizza
layout: post
permalink: "/tofu-engine-4"
comments: true
categories: 
  - tofu-engine
  - devlog
tags: 
  - render
  - blit
published: false
---
In the [previous](/tofu-engine-3) `#devlog` I mentioned that using a fragment-shader to implement palette shifting appeared as a potential performances bottleneck. In fact, sending a uniform array of 256 element to the fragment-shader is costly and, when called several times for each frame, makes the FPS drop rapidly. In addition to that, it occurred to me that pixel-perfect primitives are difficult to implement in OpenGL. Among the many, smaller circles (e.g. with a two pixel-wide radius) are almost impossible to achieve.

That has been more than enough to convince me to jump and write an in-house (almost) completely software renderer.

*Which was something already tinkered about, lately*

Switching from using OpenGL to a software *blit* functions was pretty straight-forward. The engine was already modularized enough and the *graphic layer* was self-contained and exposed a generic API. Also, since I've been code-pulling pixels since the mid-'90s, I already knew the subject. However, I tried to apply some more advanced features (such as independent x/y scaling, rotations, affine transformations) (that in the past I just skipped due to ). Also, I wrote basic 2D primitives, which proved a bit more challenging.

During the whole process, I struggled to keep the code as consistent, clean, and optimized as possibile without sacrificing understandability. In the end I'm pleased with the result.

For smaller canvases (e.g. 256x256) I gained almost a **x8** performance boost over the OpenGL version of the renderer (when switching palette and applying shifting). Big canvases, or when not using the advanced palette feature, the performance dropped a bit. However, this is not dramatic... and let's not forget that the engine is primarily aimed to *lo-fi* games.

The current features after this iteration

* explicit screen clear operation;
* blitting operations with independent x/y scaling, rotation (w/ customizable rotation *anchoring*), and x/y flipping;
* SNES-inspired [Mode-7](https://en.wikipedia.org/wiki/Mode_7) affine transformation w/ scan-line tweakable parameters (table based, much like HDMA),
* per-color palette shifting and transparency (applied on each draw operation);
* rectangular clipping for blitting and primitives drawing;
* drawing-state (clipping-region, palette, shifting, transparency) push/pop;
* multi-level FPS capping.

While I was on the topic, I drafted some features that I'm still not sure if they are worth to be included, that is

* pattern-based fill for primitives;
* raster operations;
* global foreground (i.e. pen) color.

The GLFW/OpenGL layer is used only to present the final framebuffer data to the user (by converting the 8-bit offscreen buffer to ARGB and moving to a OpenGL texture with a `glTexSubImage2D` call and a final stretch-drawing the texture as triangle-strip). The only advanced feature I kept is the chance to provide a *post-fx* fragment-shader.

This engine iteration has also been the change to optimize Lua's integration. I've decided to favour, internally, 32 bit *floats* and *integers* and Lua has been compiled with such settings. Also, I've perfected the build process by adding automatic `luacheck` and generation of the embedded scripts. Finally, I added both a basic memory-leak checker and `valgrind` (which proved extremely useful to track a nasty bug... but one need to be careful and enabled Mesa software rendering or the application will stop abruptly).



TODO: pre-configured perspective/barell/other mode7 effects.

Pulling pixels is something I always liked to to. Since the mid-'90s, when  I've been writing several libraries/APIs (mostly for personal use)

In 1995 (circa) I put my hands on my first PC. I've been using Commodore home-computer (C64 first, then the Amiga) since my early childhood, and my programming experience was based on them.

Of course my first interest was how to pull pixels on the screen and VGA's Mode-13h was something really easy to understand and use. After years of using some very elaborated and custom chips (I'm talking about you, VIC-II and Agnus) with very specific and peculiar features, facing something so simple and w/o any interesting feature was... strange.

Not only I had to re-learn pretty much everything about a platform (the PC) which seemed to devoid of features, but I also had to learn how to implement every graphics feat I needed.

