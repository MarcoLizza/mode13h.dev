---
title: 'The making of a 2D Software Blitter #1'
author: marco.lizza
layout: post
permalink: "/the-making-of-a-2d-software-blitter-1"
comments: true
categories:
  - tofu-engine
  - devlog
tags:
  - 2D graphics
  - algorithms
  - technical
---
Some days ago, a fellow **#gamedev** (hi, [Mathew](https://twitter.com/mattymariani)! :)) posed me a interesting question. He was interested in investigating with more detail the algorithms, technologies, and raster algorithms in general... and asked me for a direction on a book on the subject. To my knowledge, such a book doesn't exist. Moreover, most of the information on the subject is very difficult to dig from the Internet. Back in the last '90s and early '00s one could find long precise dissertations on many raster algorithms (they were the backbone of pretty much every *Mode 13h* game of the VGA era). But nowadays? Sadly most of this information has been lost.

One good source of information is the source code for some multimedia libraries (like [SDL](https://www.libsdl.org/) and [SFML](https://www.sfml-dev.org/), just to name two of the most famous), but it does take time (and a good amount of effort) to extract the core logic, as the code is convoluted due to optimizations.

Long story short: since in the last two-years-and-a-half I've been working on [**#tofuengine**](/tofu-engine) (but if you are reading this page probably you already know that) why don't we take the chance to describe some of its internals in a more detailed fashion? So, I took out my notebook of scribbled a potential outline of the algorithms I'd like to present. Among the many I found that I'd like to describe things like:

* blitting
  * fast-copy
  * sprite-sheets
  * clipping
  * transparency
  * shifting
  * scaling
  * rotations
  * shifting
  * stencils
  * blending
* exotic operations
  * software fragment shaders
  * copperlists emulation
  * mode7
  * pixel-perfect-collision
* serialization
  * loading and palettizing an image
  * transferring to the framebuffer
* optimizations
  * dirty-rects
  * LUTs
  * SIMD-like operations
  * working with RGBA pixels

Without further ado, let's get started!

## Pixel format

First things first: how we are going to represent our images?

In **#tofuengine** every image is [palettized](https://en.wikipedia.org/wiki/Indexed_color). That is, every pixel maps to a byte, which in turn references an entry into a **CLUT** (*C*olor *L*ook-*U*p *T*able, or **palette**, holding the actual [RGB](https://en.wikipedia.org/wiki/RGB_color_model) color for every pixel).

> Having used a byte to represent a single pixel, as a consequence our palette can have **at most** 256 distinct colors. Could we use 16 bits to have larger palettes? Sure... but handling palettes of up to 65336 entries would be unpractical (other than carrying performance drawbacks in the way the would impact the CPU cache).

This is a frame-buffer organization variant that was common in the '90s and was colloquially called **chunky pixel** (more formally, **packed pixel** but this second one is a more broad term), in contrast with **planar pixels** organization. Each of the two organizations has advantages and disadvantages, and none of the two can be determined and the *best* one... but, honestly, chunky pixels make writing a software-renderer easier! :)

Since a raster image is bi-dimensional are we required to use bi-dimensional arrays?

```c
#include <stddef.h>
#include <stdint.h>

typedef uint8_t pixel_t;

int main(int argc, const char *argv[])
{
    const size_t width = 320;
    const size_t height = 240;
    pixel_t image[height][width]; // Row-major order, array indices are row-first.

    image[17][23] = 42; // Set pixel at `<23, 17>` to `42`.

    return 0;
}
```

We could. But on the long run, when we are going to work with a dynamically allocated image, would be very awkward. We are going to use a uni-dimensional array and a bit of math (since, as a general rule, every N-dimensional array can be flattened and mapped to a 1-dimensional array).

```c
#include <stddef.h>
#include <stdint.h>

typedef uint8_t pixel_t;

int main(int argc, const char *argv[])
{
    const size_t width = 320;
    const size_t height = 240;
    pixel_t image[width * height]; // We'll retain row-major order as access.

    image[17 * width + 23] = 42; // Set pixel at `<23, 17>` to `42`.

    return 0;
}
```

## Blitting a Sprite

Or, said in other terms, copying a bidimensional region of an image to another image. **Blitting** (short for **block transfer**) is a generic, *old-skool*, term that means "copy". A **sprite** is a general term that indicates a moving object (or, at least, something that is put on a second time) over a background.

```c
#include <stddef.h>
#include <stdint.h>

typedef uint8_t pixel_t;

typedef struct image_t {
    size_t width, height;
    pixel_t *pixels;
} image_t;

// We expect these two functions to be defined somewhere into some other translation-unit.
extern image_t *image_create(size_t width, size_t height);
extern void image_destroy(image_t *image);

static void _blit(image_t *destination, int x, int y, const image_t *source)
{
    pixel_t *dptr = destination->pixels + y * destination->width + x;
    const pixel_t *sptr = source->pixels;

    const int dskip = destination->width - source->width;

    for (int height = source->height; height; --height) {
        for (int width = source->width; width; --width) {
            *(dptr++) = *(sptr)++;
        }
        dptr += dskip;
    }
}

int main(int argc, const char *argv[])
{
    image_t *background = image_create(320, 240);
    image_t *sprite = image_create(16, 16);

    _blit(background, 23, 17, sprite); // Draw `sprite` on the `background` with upper-left corner at `<23, 17>`.

    image_destroy(sprite);
    image_destroy(background);

    return 0;
}
```

The logic in the `_blit()` function is simple: seek the initial to-be-written address in the destination buffer, then proceed in row-major order and copy the source data, skipping to the beginning of the next row every time a row is copied (for a long time I used to call the `dskip` amount *modulo* as a nod to Amiga's `BLTxMOD` registers which fulfilled this exact role at a hardware level during [Blitter](https://en.wikipedia.org/wiki/Blitter)'s operations).

We are not doing anything too fancy, for the moment. Also we are performing any check and/or clipping at all. For that reason, the `source` image needs to be smaller than `destination`, and copying the sprite over the background boundaries would corrupt the result (and even cause a segmentation-fault in some cases).

## Next to come

In the next entry we'll see how to better arrange and handle multiple sprites on a *sprite-sheet* (also called *banks*, *maps*, or *atlases*). Then we spice things up a bit by adding more advanced features to out **Blitter**.
