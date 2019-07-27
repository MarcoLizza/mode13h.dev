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

Next in the list animations, stencil-drawing, and gaming-controller support need to be implemented. In the meanwhile, I'll consider if it's worth switching from **Wren** to some other more common language (Lua or JavaScript). For the moment I don't need any script-level debugging but as the projects grow in size it will soon become a major requirement. Also, better IDE integration and support would be definitely a plus.

## RICS or CISC?

This is something I bang my head against pretty much every time. While adding functionality to an SDK/API/engine I reach a point where the basic (simple) bricks are present (e.g. drawing primitives). Should I keep on adding more complex features or left them as dependant on the game implementation?

An example of this is animation support.

Creating a sufficiently generic animation support might require quite a bit of time and eventually some features could still be missing. Also, most of the times a very simple animation system might be more than enough.

Since **#tofuengine** is, as its name says, an *engine* ad not a *framework* I'll stick to this and refine the engine as much as possible. The game instance shuold find (almost) everything it needs natively in the engine and no (or at least very few) code should be written to cope for what is missing.

This lead also to another subject, that is offering support for the typical tools that one whould need while writing a game (vector classes, basic physics, etc...).

## Demo time!

Every time I add a new feature to the engine I write a simple demo project to test it out. There are times, however, when I just want to try a technique and see how it renders (this is something I also occasionally do with my [LOVE2D workouts]()).

Recently **sprite-stacking** is a techique that has rapidly become the current trend for most games. A bunch of nice games using it has been published (...) and some *drawing* tools has been developed (with spritepile and spritestack become a feud).

Personally, while the final effect rotating of the technique is intriguing is mostly think it is really wasteful. The paint process just renders layers that are continuously overdrawn. Also animation support is a bit of a hassle and authoring as well is very tedious. Apart the aforementioned tools, [MagicaVoxel]() is easy enough to be a valid choise.

> I think the technique is not that bad as it was the pre-rendered 3D sprites during the '90s (the lack of alpha trasparency made them cringe) but it just can't stand the comparison with hand-drawing sprites.

That been said, *sprite-stacking* can be the chance for me to implement a clone of one the games I loved as a kid: Atari's [Super Sprint](). That game was awesome. Some years later, on the Amiga, SkidMarks has been probably the best game on the genre. Coding a simple single-screen inspired game should be fun and the occasion to add what is missing to the engine. No scrolling, for the moment, since I would equire some changes in the engine (e.g. the support for an offscreen canvas).

Added Vecto2D class, addedd RigidBody class. Should add some other internal ready-to-use classes?
