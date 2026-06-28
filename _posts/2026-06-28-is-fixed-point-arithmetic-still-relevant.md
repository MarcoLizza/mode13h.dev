---
title: Is Fixed-Point Arithmetic Still Relevant?
layout: post
permalink: /is-fixed-point-arithmetic-still-relevant/
categories:
  - tofu-engine
  - devlog
tags:
  - engine
  - math
  - optimization
---
Over the last six months I've been polishing and optimizing [Tofu Engine](https://tofuengine.org/)'s software renderer. Yeah, I know... a full and detailed devlog on this topic post is due, but while I'm organizing my thoughts on that, here's an interesting byproduct the I think we shall talk about. :)

After quite a bit of rework to the [blitter](https://en.wikipedia.org/wiki/Bit_blit) code, the hot-loop part of the routines (namely, the (roto)scaler) have been rewritten as incremental. We start from a source coordinate, add a constant step for each destination pixel, sample the source texture, and move on. You know the drill.

As a basic arithmetic type I've been using `float` since the very beginning, mostly on the assumption that modern CPUs (with their FPUs) and compilers are clever enough to operate on this type with roughly the same performance as integer types.

But are we that this assumption is correct?

While iterating through the destination area, every pixel coordinate (in the source texture) is computed as a `float` and then converted to `int` in order to sample the source texture pixel.

At that point, a few questions naturally appeared. Is the approximation done with `float` precise enough for this use case? Are we introducing unnecessary rounding noise while repeatedly accumulating small increments? How much does the float-to-int conversion cost? And, since we are in an accumulation hot-loop anyway, would a DDA-like approach be preferable?

I rapidly threw together some simple benchmarks and the results were... surprising. Floating point is, as expected, not the silver bullet we might think it is.

This led me to write a small utility library to pack the fixed-point functions I needed (yeah, I know that some other nice libraries exists, but you know... the thrill or reinventing the wheel). I chose for a [`Q16:16`](https://en.wikipedia.org/wiki/Q_(number_format)) format as the default, since the values I'll be using won't reasonably exceed the `int16_t` range. Having 16 bits of fractional part still gives enough subpixel precision, while keeping the integer part large enough for ordinary work. Addition and subtraction remain integer operations and conversion back to an integer pixel coordinate is basically a bit-shift.

I won't be using it everywhere across my codebase, 'though, especially to replace what naturally lives in floating point. Floating point math is perfectly suited for a lot of higher-level calculations. Also, I would avoid using it where values can easily escape the range above, as the library does not provide saturation or overflow protection. I'm aiming for is more of a hybrid strategy: do the comfortable high-level setup in `float`, convert stable starting values and increments once, then let fixed point handle the hot inner portion where it actually buys something.

You can find the library [here](https://github.com/MarcoLizza/libfix32), along with the relevant benchmark results.

That is all for now. See ya next. :)
