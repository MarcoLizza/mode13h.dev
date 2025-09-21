---
title: Reworked Image Loading
layout: post
permalink: /reworked-image-loading/
categories:
  - tofu-engine
  - devlog
tags:
  - engine
  - image
  - optimization
---
A crucial concept behind the design of **Tofu Engine** is the idea of using *palettized* graphics. All image assets are PNG images, which are a popular choice for many other engines due to their lossless nature and wide support across graphics tools.

During the loading process, an image is automatically converted from its original format (e.g. 32-bit ARGB). A reference palette (with a variable size of up to 256 colours) is used, and the  [best matching colour](https://en.wikipedia.org/wiki/Color_difference) is computed for each pixel and stored into an 8-bit-per-pixel buffer. The value of each pixel in this buffer is an index to a palette entry (also known as a CLUT, or colour look-up table).

Sounds simple enough, right? In practice, it's that simple, and the initial implementation of the engine loading routine was as follows:

1) Load the PNG image into a 32-bit, `WxH`-sized buffer (A).
2) Allocate an 8-bit-per-pixel buffer B of size WxH.
3) For every pixel in buffer A, find the best matching colour in the palette and store it in buffer B.
4) Free buffer A.

With the help of some support libraries (namely, [stb](https://github.com/nothings/stb)) this was an easy feat. However, there were some issues:

* The full ARGB buffer is loaded only to be released almost immediately. This is a wasteful use of the heap that should be avoided.
* It was also slow. This isn't something one would notice at first since the assets of a 2D pixel art engine are usually quite small. However, when loading larger images, the conversion wasn't as fast as one would hope.

The second issue can easily be solved using [memoization](https://en.wikipedia.org/wiki/Memoization): for each ARGB colour, the best matching colour only needs to be found and stored in a hash table once. Subsequent occurrences will skip the lengthy computation and simply use the cached value. For example, loading an `800x800` image took `~0.5` seconds; with memoization, this time was reduced to `~0.1` seconds. That's quite an improvement!

Solving the first issue, however, is a different matter.

Ideally, the fastest loading time would be achieved if the original image were already in a palette-based format, as it could be loaded straight into the final buffer without any intermediate processes. However, this would contradict a fundamental design choice we made when designing the game engine: **the reference palette is controlled by the script and shouldn't be fixed**.

We can, however, reach a compromise by mean of **progressive loading** of the PNG image. By loading it row-by-row and converting each scanline on the fly we would require only an additional buffer for the row. To accomplish this we switch to [spng](https://libspng.org/).

We adapted the engine code, moving all the image loading routines to an internal module called `image` and generalising the API to be callback-driven. This allowed us to achieve our objective of not affecting loading times (we are totally on par with the previous "non-progressive" implementation), while using only a fraction of the memory.

We also added miniz as a library, offering a ZLIB-grade stream compressor that we can use for other features in the future. Also, reworking the API gave us the opportunity to refactor it, making it far more generic and abstract (thanks to the loading callbacks).

I guess we should be satisfied with our results, as we can't do any better without failing our design choices.

Or no? :)