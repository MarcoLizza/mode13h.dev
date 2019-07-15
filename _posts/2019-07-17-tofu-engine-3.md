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

In the last months **#tofuengine** saw a major overhaul of its internals: I ditched ([RayLib](https://www.raylib.com/)) in favour of [GLFW](https://www.glfw.org/). **Raylib** is really a nice library but it is too obtrusive for my purposes... in fact it is aimed to be more a full-fledged framework (and it just did this very well) rather than a media library.

> [SDL](https://www.libsdl.org/) was the other candidate library, but 

This meant that a large part of the engine has to be rewritten/integrated. However, this was definitely for the better. Now I have more control on every part of the engine rendering pipeline. For the moment, **#tofuengine** requires OpenGL 2.1 only, which should be safe enough to be supported by most platforms (and I don't aim to support mobile platforms).

All considered, the switch wasn't particularly difficult. There are a number of helper libraries (such as [GLAD]() and [stb]()) that makes the things easier.

The current features are

* palettized rendering,
* customizable post-fx shaders,
* sub-pixel drawing support,
* 

Next, a stencil-drawing and gaming-controller support should be implemented. Then, we'll consider if it's worth switching from Wren to some other more common language (Lua or JavaScript).

## Sprite-stacking

The technique is wasteful, since the layers are continuously overdrawn. However, the rotation support is nice. Authoring is more difficult, but [MagicaVoxel]() is available and easy enough.

SkidMarks clone, single screen only. Scrolling will require some changes in the engine (e.g. the support for an offscreen canvas).

Added Vecto2D class, addedd RigidBody class. Should add some other internal ready-to-use classes?