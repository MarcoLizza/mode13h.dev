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
published: false
---
In the last 4 months there have been a lot of work to refactor #tofuengine. At the moment, the internal tech has moved from *RayLib* to *GLF3*, and from *Wren* to *Lua*.

Apart this, I've also been thinking a lot about the design of the API. While it's tempting to exploit the full OpenGL features to make the engine as-cool-as-it-can-be (alpha-transparency, fragment shaders, etc...), this would denaturate its "soul". It's a *pixel-art-engine*, and those are things that weren't available in the *Golder Era of Pixel-Art* (i.e. the *16-Bit Machines Age*). At the same time, I don't want the engine to support blitting only, but to easily enabled cool pixels effects.

The palette is only the starting point for this. It can

https://hackernoon.com/pico-8-lighting-part-1-thin-dark-line-8ea15d21fed7

By no means #tofuengine wants to be a *clone* of Pico-8. The rationale is quite different, since I'm not aiming to create a *virtual console* (which is a neat thing to do, anyway). Anyhow, Lexaloffe's child is definitely an inspiration, not only due to its peculiar style and API.

Thanks to [Paul](liquidreams) (he's a greate Pico8 developer and you should definitely check its creations) I got some inputs on how Pico8 approaches some effects.

The first interesting thing is *palette re-indexing* (or *shifting*) and how *color transparency* is achived.

With the current shader-based palette implementation, adding per-color shifing and transparency is really easy to add. The current shader state is used for each drawing operations, so it's not a global configuration (perhaps will add it later on, but having a fully-customizable palette probably renders it useless).

Unfortunately, passing 256 entries long uniform arrays proves to be very expensive. So, I added some optimization by precalculating the shader program uniform locations and reuse those indexes when sending data. Also, I reduced the maximum palette length to 64 entries (256 are probably too much to be easily used). This resulted in a x3 performance boost.

The another idea came into mind: since (apparently) the major seems to be sending the uniform array to the shader, why don't we just base the whole technique on a 256x2 texture? Instead of accessing an array of values we would peek from a texture the information we need. That way we would be sending to the shader only the texture-id once upon initialization. Later we would *just* be updating the texture content and send *no* data to the shader. Clever! Smart! Cool! But also **very** wrong! In order to update a texture we need to access it content directly. One way to do it is by keeping and changing a memory buffer and use to update the texture by calling the `glTexSubImage2D()` function... which is dead slow, having to first change the current used texture (`glBindTexture()`), then move data from RAM to VRAM, and (possibly) convert it to the video-card internal representation (`GL_RGBA` or `GL_BGRA`). Even with a teeny-tiny texture of 256x2 pixel this can take quite a fraction of a frame time, and since we would be using this feature to implement *palette-shifting* (i.e. potentially change the colors re-indexing before each draw operation) frame-rate would drop significantly.

So, for the moment we are sticking to uniform arrays.

However, it's tempting to implement a software renderer and rely upong OpenGL only to move the move the data to the framebuffer. Implementing by hand the primitives would be fun, and would solve some OpenGL's quircks (for example, I won't need any *Framebuffer Object* but a single offscreen texture that is drawn, streched, and post-processed only just a moment prior buffer swapping). It might be a nice proof-of-concept. The display area is meant to be small by nature (512x512 at most).
