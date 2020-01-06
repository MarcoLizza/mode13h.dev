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
  - software-renderer
  - blitter
---
In the [previous](/tofu-engine-3) `#devlog` I mentioned that using a fragment-shader to implement palette shifting appeared as a potential performance bottleneck. In fact, sending a uniform array of 256 elements to the fragment-shader is costly and, when called several times for each frame, makes the FPS drop rapidly. In addition to that, it occurred to me that pixel-perfect primitives are difficult to implement in OpenGL. Among the many, smaller circles (e.g. with a two pixel-wide radius) are almost impossible to achieve.

That has been more than enough to persuade me to code an in-house software renderer (which was something already tinkered about, lately).

Switching from OpenGL to a software *blitter* was rather straight-forward. The engine was already modularized enough and the *graphic layer* was self-contained and exhibited a generic API. Also, since I've been code-pulling pixels since the mid-'90s, I already knew the topic. Nevertheless, I tried and add some advanced features (such as independent x/y scaling, rotations, affine transformations) that in the past I just avoided since they were unneeded and/or too complex for the CPUs of the time. In addition to that, I coded some basic 2D primitives drawing routines, which proved a bit more challenging.

The [GLFW](https://www.glfw.org/)/[OpenGL](https://www.opengl.org/) layer is used only to present the final frame-buffer data to the user. This is achieved by converting *on-the-fly* the 8-bit offscreen buffer to an `ARGB` memory buffer and moving the latter to a pre-allocated (OpenGL) texture with a single `glTexSubImage2D` call. With a single triangle-strip, the texture is drawn onto the video frame-buffer, stretching if necessary. This also enabled the presence of an (optional) single *post-fx* fragment-shader (which is the only advanced feature I kept).

During the whole process, I struggled to keep the code as consistent, clean, and optimized as possible without sacrificing understandability. In the end, I'm pleased with the result.

When palette switching and/or shifting is used, for mid-to-small canvases (e.g. `256x256` pixels) I gained almost an **x8** performance boost over the OpenGL version of the renderer. With bigger or when palette switches/shifts aren't used, the performance boost is less evident. However, this is not dramatic... let's not forget that the engine is primarily aimed to *lo-fi* games (I don't plan to draw ~10K sprites regularly). With some kind of z-index or dirty-region technique we *could* possibly have better performances, but the additional complexity won't make it worth the effort.

We can sum up the engine features added with this iteration as follows:

* explicit screen clear operation (this was implicit in each frame-buffer redraw);
* support for *BOB* (**B**litter **OB**ject) drawing with independent x/y scaling, x/y flipping, rotation, and rotation *anchoring*;
* SNES-inspired [Mode-7](https://en.wikipedia.org/wiki/Mode_7) affine transformation w/ scan-line dependent parameters (table-based, much like [HDMA](https://wiki.superfamicom.org/grog's-guide-to-dma-and-hdma-on-the-snes)),
* per palette-index shifting and transparency (applied on each draw operation);
* rectangular clipping for both *BOB*s and primitives drawing;
* push/pop operations for the current drawing-state (clipping-region, palette, shifting, transparency);
* multi-level auto-adaptive FPS capping.

While on the topic, I drafted some features that I'm still not sure if they are worth to be included, that is:

* pattern-based fill for primitives;
* raster bitwise operations;
* global foreground (i.e. pen) color, implicitly used when not specified.

This engine iteration has also been the chance to optimize Lua's integration. I've decided to favour, internally, 32 bit *floats* and *integers* and Lua has been compiled with such settings (I don't feel the need to use larger data-types, honestly). Also, I've perfected the build process by adding automatic `luacheck` and generation of the embedded scripts (based on `hexdump`... I'd like to pre-compile the Lua files to byte-code, too). Finally, I added a memory-leak checker (i.e. [stb_leakcheck.h](https://github.com/nothings/stb/blob/master/stb_leakcheck.h)) and `valgrind`. The latter proved extremely useful to track a nasty bug (but one needs to be careful and enable Mesa software rendering or the application will stop abruptly).

For the next iteration, I plan to move the first steps into the audio subsystem (first of all by finding a suitable portable audio library to leverage). Also, I'd like to write some separate blog entries to detail the algorithms used in coding the software-rendered (this ancient art is rapidly disappearing).
