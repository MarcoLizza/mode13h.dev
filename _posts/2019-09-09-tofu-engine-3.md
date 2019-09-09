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
  - pico8
  - palette
  - effects
---
During the last 4 months, #tofuengine has seen a lot of improvements. Under-the-hood the engine have been refactored to switch from *RayLib* to *GLF3* and from *Wren* to *Lua* (more on these two subjects on separate #devlog entries).

Also, I've been thinking about the engine's API, design, and features. While it's tempting to exploit the OpenGL (or, to be precise, modern GPUs) to make the engine as-cool-as-it-can-be (alpha-transparency, fragment shaders, etc...), this wouldn't fit the engine's "soul". It's a *pixel-art-engine*, and those advances effects weren't available in the *Golder Age of Pixel-Art* (i.e. the *16-Bit Machines Age*). Nevertheless, at the same time, I don't want the engine to support image-blitting only but enable also pixels effects.

Having a palette-based rendering does offer, for example, palette cycling... but that's pretty much all! To implement, for example, *line-of-sight* lighting in a rouge-like one would be duplicating the assets and draw the same tile with different lighting levels. Also, displaying a *blink-when-hit* effect would require an explicit asset. We could also change the palette dynamically before a drawing operation, but this can rapidly become messy.

So, I asked fellow developer [Paul Nicholas](https://www.liquidream.co.uk/) (who happens to be a great [PICO-8](https://www.lexaloffle.com/pico-8.php) developer, you should check its creations) how those effects and issues are approached ([this](https://hackernoon.com/pico-8-lighting-part-1-thin-dark-line-8ea15d21fed7) has been another source of inspiration).

> Disclaimer: by no means, #tofuengine wants to be a PICO-8's *clone*. I'm not aiming to create a *virtual console* (which is a neat thing to do, anyway) but a simple and effective lo-fi game-engine. However, its peculiar style and minimalistic API are something to be inspired by.

Out of this chit-chat, the first interesting features that were elaborated are **palette re-indexing** (or **palette-shifting** if one prefers) and **color transparency** are achieved. Until now in #tofuengine the current palette affects any successive drawing operation. To change dynamically the color of a tile, for example, one needs to change the palette then eventually revert the change. This isn't something difficult, but can rapidly become messy over time. As for transparency, this is defined with the alpha channel in the sprite/tile atlas.

With some minor changes to the palette shader, I've added two tables (i.e. arrays) to define the color re-indexing and transparency. Easy-peasy. Now real-time color substitution and transparency are a reality!

> At the moment, the current shader state is used for each drawing operations but it's not a **global post-draw** setting. Perhaps I will add it later, but having a fully-customizable palette probably renders it useless.

![Palette Effects](/assets/videos/palette-effects.gif)

Unfortunately, yep there's a black cloud in every silver lining (!), passing 256 entries long uniform arrays proves to be expensive when changing it 50 times for frame (for example). With some optimization, such as precalculating the shader program uniform locations and reducing the maximum palette length to 64 entries, I gained a x3 performance boost. Still, I don't like it the whole idea of sending a continuous stream of data from the CPU to the GPU to feed the shader.

Idea! Why don't we just base the whole technique on a 256x2 texture? Instead of accessing an array of values we would peek from a texture the information we need. That way we are required to send to the shader just the texture-id once during the initialization. Later on, we'll be periodically updating the texture content without explicitly sending data to the shader. Clever! Smart! Cool! But also **very** wrong! Updating a texture requires direct write-access to its content. One way to do it is by working on a memory buffer and use to update the texture by calling the `glTexSubImage2D()` function... which can't be blazing fast, having transfer data chunks from RAM to VRAM, and (possibly) convert it to the video-card internal representation (`GL_RGBA`, `GL_BGRA`, or whatever). Even with a teeny-tiny texture of 256x2 pixels takes a significant fraction of the frame slice, and since we would be using this feature to implement *palette-shifting* (i.e. potentially change the colors re-indexing before each draw operation) frame-rate would drop significantly.

So, for the moment we just are sticking with using uniform arrays.

However, it's tempting to implement a software renderer and rely upon OpenGL only to move the final processed data to the framebuffer. Thant way, palette operation would be implemented in the heap and any kind pixel read-and-write operation won't require CPU-to-GPU boundary-crossing. This would open to door to many tricks such as

* pixel bitwise operations,
* custom fill-patterns,
* real-time stencil clipping (a-la Commodore Amiga),
* sub-tile offsetting (a-la Commodore 64),
* etc...

There won't be any need for a temporary *Framebuffer Object*, but a single offscreen texture that would be updated, drawn, stretched, and post-processed only during the end-of-frame flip/swap. The graphics abstraction layer would simplify a lot. Also, since the virtual display is not meant to be typically very large (640x480 pixels at most), the RAM-to-VRAM transfer shouldn't be too costly.

Most of all, implementing such a software renderer would be a lot of fun! :)