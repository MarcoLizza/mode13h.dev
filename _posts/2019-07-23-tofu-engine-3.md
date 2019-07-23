---
title: 'tofu-engine #3'
author: marco.lizza
layout: post
permalink: "/tofu-engine-3"
comments: true
categories: 
  - tofu-engine
  - devlog
tags: 
  - raylib
  - glfw
  - sdl
  - sprite stacking
published: false
---
## Refactoring the rendering pipeline

In the last months **#tofuengine** saw a major overhaul of its internals: ([RayLib](https://www.raylib.com/)) has been abandoned in favour of [GLFW3](https://www.glfw.org/). **Raylib** is really a nice library with a lot of amazingly easy-and-ready-to-use functions. it has almost everything you'll need to write a game but it is *too obtrusive* for my purposes... in fact it is aimed to be more a full-fledged framework (and it just does this very well) rather than a media library.

> [SDL](https://www.libsdl.org/) was the other candidate library, but from my point of view it 

This meant that a large part of the engine has to be rewritten to integrate **GLFW3**, and some missing part (e.g. the logging sub-system) needed to be coded from scratch. However, this was definitely for the better since now I way have more control over the engine rendering pipeline. Also, general performances For the moment, **#tofuengine** requires *OpenGL 2.1* or better, which seems to be supported by most platforms (I don't aim to support mobile devices).

All considered, the switch wasn't particularly difficult since a number of helper libraries (such as [GLAD]() and [stb]()) made the things easier.

Among the features we currently find

* automatic window scale w/ full-screen support,
* palette-based rendering w/ alpha transparency and automatic image re-indexing (nearest color match),
* sprite-sheet based rendering with scaling and x/y flipping support.
* Default bitmap font in 5 different sizes (credits to [Frederic Cambus](https://www.cambus.net/spleen-monospaced-bitmap-fonts/)),
* programmable post-fx shaders,
* sub-pixel drawing support,
* keyboard gamepad-like input,
* scripting callback timer,
* priority-controlled colored console logging.

Next in the list animations, stencil-drawing, and gaming-controller support need to be implemented. In the meanwhile, I'll consider if it's worth switching from **Wren** to some other more common language (Lua or JavaScript). For the moment I don't need script-level debugging but as the projects grow in size it will be a major requirement. Also, better IDE integration and support would be definitely a plus.

## Sprite-stacking

The technique is wasteful, since the layers are continuously overdrawn. However, the rotation support is nice. Authoring is more difficult, but [MagicaVoxel]() is available and easy enough.

SkidMarks clone, single screen only. Scrolling will require some changes in the engine (e.g. the support for an offscreen canvas).

Added Vecto2D class, addedd RigidBody class. Should add some other internal ready-to-use classes?