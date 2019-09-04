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
https://hackernoon.com/pico-8-lighting-part-1-thin-dark-line-8ea15d21fed7

By no means #tofuengine wants to be a *clone* of Pico-8. The rationale is quite different, since I'm not aiming to create a *virtual console* (which is a neat thing to do, anyway). Anyhow, Lexaloffe's child is definitely an inspiration, not only due to its peculiar style and API.

Thanks to [Paul](liquidreams) (he's a greate Pico8 developer and you should definitely check its creations) I got some inputs on how Pico8 approaches some effects.

The first interesting thing is *palette re-indexing* (or *shifting*) and how *color transparency* is achived.

With the current shader-based palette implementation, adding per-color shifing and transparency is really easy to add. The current shader state is used for each drawing operations, so it's not a global configuration (perhaps will add it later on, but having a fully-customizable palette probably renders it useless).

Unfortunately, passing 256 entries long uniform arrays proves to be very expensive. So, I added some optimization by precalculating the shader program uniform locations and reuse those indexes when sending data. Also, I reduced the maximum palette length to 64 entries (256 are probably too much to be easily used). This resulted in a x3 performance boost.

However, it's tempting to implement a software renderer and rely upong OpenGL only to move the move the data to the framebuffer. Implementing by hand the primitives would be fun (and would solve some OpenGL's quircks). It might be a nice proof-of-concept. The display area is meant to be small by nature (512x512 at most).
